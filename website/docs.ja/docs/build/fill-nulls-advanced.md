---
title: "メトリックにnull値を入力する"
id: fill-nulls-advanced
description: "ワークフローのモデリングなど、dbt セマンティック レイヤーと MetricFlow の高度なトピックについて学習します。"
sidebar_label: Fill null values for metrics
---

メトリクス内のnull値を埋める戦略を理解し、実装することは、正確な分析を行うための鍵となります。このガイドでは、データの完全性を確保し、エンドユーザーがより情報に基づいた意思決定を行い、dbtワークフローを強化するための「fill_nulls_with」と「join_to_timespine」について説明します。

### null値について

`fill_nulls_with` を使用すると、指標内の null 値を 0 などの値（または任意の整数）に置き換えることができます。これにより、すべてのデータ行に数値が表示されます。

このガイドでは、指標に null 値が含まれていないことを確認する方法について説明します。

- `simple`、`cumulative`、`conversion` 指標には `fill_nulls_with` を使用します。
- 派生指標と比率指標には `join_to_timespine` と `fill_nulls_with` を併用し、null 値が表示されないようにします。

### シンプルな指標の場合は null 値を埋める

例えば、サイト訪問はあったもののリードが発生しなかった日を処理したい場合、`fill_nulls_with` を使用することで、コンバージョンが発生しなかった日のリードの値を 0 に設定できます。

3 つの指標があるとします。

- `website_visits` と `leads`
- そして、サイト訪問数に対するリード数の比率を計算する `leads_to_website_visit` という派生指標です。

コンバージョンがない日には、リード メトリックのメジャー入力に `fill_nulls_with` パラメータを追加して、リード値をゼロに設定できます:

<File name='models/metrics/website_vists.yml'>

```yaml
metrics:
  - name: website_visits
    type: simple
    type_params:
      measure:
        name: bookings
  - name: leads
    type: simple
    type_params:
      measure:
        name: bookings
        fill_nulls_with: 0 # This fills null values with zero
  - name: leads_to_website_visit
    type: derived
    type_params:
      expr: leads/website_visits
      metrics:
        - name: leads
        - name: website_visits
```

</File>

`website_visits` および `leads` メトリックには次のデータがあります:

| metric_time | website_visits |
| --- | --- |
| 2024-01-01 | 50 |
| 2024-01-02 | 37 |
| 2024-01-03 | 79 |


| metric_time | leads |
| --- | --- |
| 2024-01-01 | 5 |
| 2024-01-03 | 8 |
* `leads` メトリックには `2024-01-02` のデータが存在しないことに注意してください。

訪問がない日はないものの、リードがない日もあります。`leads` 指標に `fill_nulls_with: 0` を適用し、これらの指標をまとめてクエリすると、コンバージョンがない日のリードはゼロと表示されます。

| metric_time | website_visits | leads |
| --- | --- | --- |
| 2024-01-01 | 50 | 5 |
| 2024-01-02 | 37 | 0 |
| 2024-01-03 | 79 | 8 |

### 派生メトリックと比率メトリックには join_to_timespine を使用する

他のメトリクスから計算されたメトリクスについて、毎日および日次のカバレッジの完全なデータセットを確保するには、`join_to_timespine` を使用して `derived` メトリクスと `ratio` メトリクスの null 値を埋めることができます。これらのメトリクスは、直接のメジャー（生データ）ではなく、他のメトリクス（他の計算）から構築されるため、MetricFlow ではメトリクスをレンダリングするために追加のサブクエリレイヤーが必要になります。サブクエリのネストは次のとおりです。

- `derived` メトリクスと `ratio` メトリクスの場合、サブクエリのネストは 3 段階（派生または比率メトリクス → 入力メトリクス → 入力メジャー）です。
- `simple` メトリクスと `cumulative` メトリクスの場合、サブクエリのネストは 2 段階（シンプルまたは累積メトリクス → 入力メジャー）のみです。

`coalesce` は `derived` または `ratio` メトリクスの 3 番目のサブクエリレイヤーには適用されないため、最終的な結果セットに null が含まれる可能性があります。

* データが存在しない場合でも、すべての日付の行を含めたい場合は、メジャー入力を受け取るメトリクスでも `join_to_timespine` を使用できます。

### 派生メトリックと比率メトリックの null 値を埋める

派生メトリックと比率メトリックの null 値を埋めるには、タイムスパインを使用してリンクすることで、日次データの範囲を確保できます。[前のセクション](#use-join_to_timespine-for-derived-and-ratio-metrics) で説明したように、これは `derived` 指標と `ratio` 指標が *measures* ではなく *metrics* を入力として取るためです。

例えば、次の構造では、`derived` 指標の最終指標計算において、外側の 3 番目のレンダリング レイヤーで `COALESCE` が適用されないため、最終結果 (`leads_to_website_visit` 列) に null が残ります。

| metric_time | bookings | leads | leads_to_website_visit |
| --- | --- | --- | --- |
| 2024-01-01 | 50 | 5 | .1 |
| 2024-01-02 | 37 | 0 | null |
| 2024-01-03 | 79 | 8 | .1 |

`2024-01-02` の `leads_to_website_visit` にゼロ値を表示するには、`leads` 指標をタイムスパインモデルに結合し、各日の値を確保する必要があります。これは、`leads` 指標設定の `measure` パラメータに `join_to_timespine` を追加することで実現できます:

<File name='models/metrics/leads.yml'>

```yaml
- name: leads
  type: simple
  type_params:
    measure:
      name: bookings
      fill_nulls_with: 0
      join_to_timespine: true
```
</File>

これを実行すると、タイムスパイン結合後に `leads` メトリックをクエリすると、各日のレコードが作成され、null 値はすべてゼロで埋められます。

| metric_time |  leads | leads_to_website_visit |
| --- | --- | --- |
| 2024-01-01 |  5 | .1 |
| 2024-01-02 | 0 | 0 |
| 2024-01-03 |  8 | .1 |

ここで、`derived` メトリックでメトリックを組み合わせると、`2024-01-02` の `leads_to_website_visit` の値はゼロになり、最終的な結果セットには null 値が含まれなくなります。

## FAQs

<Expandable alt_header="複数のテーブルの上に定義された派生メトリックの null 値を処理する方法">

複数のテーブルのデータを使用する派生メトリックで null 値を処理する方法の追加の例と説明については、[MetricFlow の問題 #1031](https://github.com/dbt-labs/metricflow/issues/1031) を参照してください。

</Expandable>

