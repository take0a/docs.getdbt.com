---
title: "マイクロバッチ増分モデルについて"
sidebar_label: "Microbatch incremental models"
description: "Learn about the 'microbatch' strategy for incremental models."
id: "incremental-microbatch"
intro_text: "マイクロバッチ増分モデルを使用して、大規模な時系列データセットを効率的に処理します。"
---

:::info

[dbt Cloud "最新"](/docs/dbt-versions/cloud-release-tracks) および dbt Core v1.9 以降でご利用いただけます。

カスタムマイクロバッチマクロを使用する場合は、`dbt_project.yml` で [distinct behavior flag](/reference/global-configs/behavior-changes#custom-microbatch-strategy) を設定してバッチ実行を有効にしてください。カスタムマイクロバッチマクロがない場合は、このフラグを設定する必要はありません。dbt は [microbatch strategy](#how-microbatch-compares-to-other-incremental-strategies) を使用するすべてのモデルに対してマイクロバッチ処理を自動的に処理します。

ディスカッションをお読みになり、ご参加ください: [dbt-core#10672](https://github.com/dbt-labs/dbt-core/discussions/10672)サポートされているアダプタのリストについては、[アダプタ別にサポートされている増分戦略](/docs/build/incremental-strategy#supported-incremental-strategies-by-adapter)を参照してください。

:::

## dbt における「マイクロバッチ」とは何ですか？

dbt の増分モデルは、前回の実行以降に追加された新しいデータまたは変更されたデータのみを変換してロードすることで、データウェアハウス テーブルを効率的に更新するように設計された [マテリアライゼーション](/docs/build/materializations) です。増分モデルでは、データセット全体を毎回再処理するのではなく、少数の行を処理し、既存のテーブルにそれらの行を追加、更新、または置換します。これにより、データ変換に必要な時間とリソースを大幅に削減できます。

マイクロバッチは、大規模な時系列データセット向けに設計された増分戦略です。
- フィルタリングのための時間ベースの範囲を定義するために、時間列（[`event_time`](/reference/resource-configs/event-time)）のみを使用します。マイクロバッチモデルとその直接の親（上流モデル）に `event_time` 列を設定します。これは、行をパーティションにグループ化する `partition_by` とは異なることに注意してください。
- マイクロバッチは、バッチ処理の効率性とシンプルさに重点を置くことで、既存の増分戦略を置き換えるのではなく、補完します。
- 従来の増分戦略とは異なり、マイクロバッチでは、[失敗したバッチの再処理](/docs/build/incremental-microbatch#retry)、[並列バッチ実行](/docs/build/parallel-batch-execution)、[バックフィル](#backfills)のための複雑な条件ロジックの実装が不要になります。

- マイクロバッチは、すべてのユースケースに最適な戦略ではないことに注意してください。信頼性の高い `event_time` 列がない場合や、増分ロジックをより細かく制御する必要がある場合など、ユースケースによっては他の戦略を検討してください。詳しくは、[`microbatch` と他の増分戦略の比較](#how-microbatch-compares-to-other-incremental-strategies)をご覧ください。

## マイクロバッチの仕組み

dbt がマイクロバッチモデルを実行する際（初回実行時、増分実行時、または指定されたバックフィル時）、設定された `event_time` と `batch_size` に基づいて、処理が複数のクエリ（または「バッチ」）に分割されます。

各「バッチ」は、単一の限定された期間（デフォルトでは 1 日分のデータ）に対応します。他の増分戦略が「古い」データと「新しい」データのみを処理するのに対し、マイクロバッチモデルでは、すべてのバッチを個別に構築または置換できるアトミック単位として扱います。各バッチは独立しており、<Term id="idempotent" /> です。

これは強力な抽象化であり、dbt がバッチを [個別に](#バックフィル)、並行して実行し、個別に [再試行](#retry) することを可能にします。

### アダプタ固有の動作

dbt のマイクロバッチ戦略は、各アダプタで利用可能な最も効率的な「フルバッチ」置換メカニズムを使用します。これはアダプタによって異なります。

- `dbt-postgres`: `merge` 戦略を使用し、「更新」または「挿入」操作を実行します。
- `dbt-redshift`: `delete+insert` 戦略を使用し、「挿入」または「置換」を実行します。
- `dbt-snowflake`: `delete+insert` 戦略を使用し、「挿入」または「置換」を実行します。
- `dbt-bigquery`: `insert_overwrite` 戦略を使用し、「挿入」または「置換」を実行します。
- `dbt-spark`: `insert_overwrite` 戦略を使用し、「挿入」または「置換」を実行します。
- `dbt-databricks`: 「挿入」または「置換」を行う `replace_where` 戦略を使用します。

詳細については、[アダプタ別にサポートされている増分戦略](/docs/build/incremental-strategy#supported-incremental-strategies-by-adapter) をご覧ください。

## 例

`sessions` モデルは、他の 2 つのモデルから取得したデータを集約し、拡充します。
- `page_views` は大規模な時系列テーブルです。多数の行が含まれ、新しいレコードはほとんどの場合既存のレコードよりも後に追加され、既存のレコードはほとんど更新されません。`page_view_start` 列が `event_time` として使用されます。
- `customers` は比較的小規模なディメンションテーブルです。顧客属性は頻繁に更新されますが、時間ベースではありません。つまり、古い顧客も新しい顧客と同様に列の値を変更する可能性があります。customers モデルでは `event_time` 列は設定されません。

結果：

- `sessions` の各バッチは、`page_views` を同等の時間制限付きバッチにフィルタリングします。
- `customers` テーブルはフィルタリングされないため、すべてのバッチでフルスキャンが実行されます。

:::tip
ターゲット テーブルに `event_time` を構成することに加えて、時間列が異なる場合でも、フィルタリングするアップストリーム モデルに対してこれを指定する必要があります。
:::

<File name="models/staging/page_views.yml">

```yaml
models:
  - name: page_views
    config:
      event_time: page_view_start
```
</File>

2024 年 10 月 1 日に `sessions` モデルを実行し、その後 10 月 2 日に再度実行します。次のクエリが生成されます。

<Tabs>

<TabItem value="Model definition">

`sessions` モデルの [`event_time`](/reference/resource-configs/event-time) は `session_start` に設定されており、これはウェブサイトにおけるユーザーのセッションの開始を示します。この設定により、dbt は複数のページビュー（それぞれが独自の `page_view_start` タイムスタンプで追跡されます）を 1 つのセッションにまとめることができます。これにより、`session_start` は個々のページビューのタイミングを、ユーザーセッション全体のより広範な時間枠から区別します。
  
<File name="models/sessions.sql">

```sql
{{ config(
    materialized='incremental',
    incremental_strategy='microbatch',
    event_time='session_start',
    begin='2020-01-01',
    batch_size='day'
) }}

with page_views as (

    -- this ref will be auto-filtered
    select * from {{ ref('page_views') }}

),

customers as (

    -- this ref won't
    select * from {{ ref('customers') }}

),

select
  page_views.id as session_id,
  page_views.page_view_start as session_start,
  customers.*
  from page_views
  left join customers
    on page_views.customer_id = customer.id
```

</File>

</TabItem>

<TabItem value="Compiled (Oct 1, 2024)">

<File name="target/compiled/sessions.sql">

```sql

with page_views as (

    select * from (
        -- filtered on configured event_time
        select * from "analytics"."page_views"
        where page_view_start >= '2024-10-01 00:00:00'  -- Oct 1
        and page_view_start < '2024-10-02 00:00:00'
    )

),

customers as (

    select * from "analytics"."customers"

),

...
```

</File>

</TabItem>

<TabItem value="Compiled (Oct 2, 2024)">

<File name="target/compiled/sessions.sql">

```sql

with page_views as (

    select * from (
        -- filtered on configured event_time
        select * from "analytics"."page_views"
        where page_view_start >= '2024-10-02 00:00:00'  -- Oct 2
        and page_view_start < '2024-10-03 00:00:00'
    )

),

customers as (

    select * from "analytics"."customers"

),

...
```

</File>

</TabItem>

</Tabs>

dbt はデータ プラットフォームに対し、各バッチ クエリの結果を取得し、同じ日のデータの `analytics.sessions` テーブルの内容を [挿入、更新、または置換](#adapter-specific-behavior) するよう指示します。この操作を実行するために、dbt は各データ プラットフォームで利用可能な「フルバッチ」置換のための最も効率的なアトミック メカニズムを使用します。詳細については、[マイクロバッチの仕組み](#how-microbatch-works) をご覧ください。

テーブルにその日のデータが既に含まれていても問題ありません。同じ入力データであれば、バッチを何度再処理しても、結果のテーブルは同じになります。

<Lightbox src="/img/docs/building-a-dbt-project/microbatch/microbatch_filters.png" title="Each batch of sessions filters page_views to the matching time-bound batch, but doesn't filter sessions, performing a full scan for each batch."/>

## 関連する設定

マイクロバッチモデルには関連する設定がいくつかあり、そのうちのいくつかは必須です:


| Config   |  Description   | Default | Type | Required  |
|----------|---------------|---------|------|---------|
| [`event_time`](/reference/resource-configs/event-time)  | 「行が発生した時刻」を示す列。マイクロバッチ モデルおよびフィルタリングする必要がある直接の親に必須です。 | N/A     |  Column  |  Required |
| [`begin`](/reference/resource-configs/begin)      | マイクロバッチモデルの「開始時刻」。これは、初期ビルドまたはフルリフレッシュビルドの開始点となります。例えば、`begin = '2023-10-01` で `2024-10-01` に日単位のマイクロバッチモデルを実行すると、366 バッチ（うるう年です！）に加えて「今日」のバッチが処理されます。 | N/A     | Date   | Required |
| [`batch_size`](/reference/resource-configs/batch-size) | バッチの粒度。サポートされている値は「時間」、「日」、「月」、「年」です。 | N/A     | String  | Required |
| [`lookback`](/reference/resource-configs/lookback)   | 遅れて到着したレコードを取得するために、最新のブックマークの前に X バッチを処理します。  | `1`     | Integer | Optional |
| [`concurrent_batches`](/reference/resource-properties/concurrent_batches) | バッチを同時実行するための dbt の自動検出をオーバーライドします。詳細については、[同時実行バッチの設定](/docs/build/incremental-microbatch#configure-concurrent_batches) を参照してください。<br />* `true` に設定すると、バッチが同時（並列）に実行されます。<br />* `false` に設定すると、バッチが順次（1 つずつ）実行されます。 | `None` | Boolean | Optional |

<Lightbox src="/img/docs/building-a-dbt-project/microbatch/event_time.png" title="The event_time column configures the real-world time of this record"/>

### 特定のアダプタに必要な構成
一部のアダプタでは、マイクロバッチ戦略のために追加の構成が必要です。これは、アダプタごとにマイクロバッチ戦略の実装が異なるためです。

次の表は、標準のマイクロバッチ構成に加えて、特定のアダプタに必要な構成を示しています。

| Adapter  | `unique_key` config | `partition_by` config |
|----------|------------------|--------------------|
| [`dbt-postgres`](/reference/resource-configs/postgres-configs#incremental-materialization-strategies) | ✅ Required | N/A |
| [`dbt-spark`](/reference/resource-configs/spark-configs#incremental-models)    | N/A | ✅ Required |
| [`dbt-bigquery`](/reference/resource-configs/bigquery-configs#merge-behavior-incremental-models) | N/A | ✅ Required |

たとえば、`dbt-postgres` を使用している場合は、`unique_key` を次のように設定します。

<File name="models/sessions.sql">

```sql
{{ config(
    materialized='incremental',
    incremental_strategy='microbatch',
    unique_key='sales_id', ## required for dbt-postgres
    event_time='transaction_date',
    begin='2023-01-01',
    batch_size='day'
) }}

select
    sales_id,
    transaction_date,
    customer_id,
    product_id,
    total_amount
from {{ source('sales', 'transactions') }}

```

この例では、`dbt-postgres` マイクロバッチが `merge` 戦略を使用するため、`unique_key` が必要です。この戦略では、データウェアハウス内のどの行をマージする必要があるかを識別するために `unique_key` が必要です。`unique_key` がないと、dbt は入力バッチと既存のテーブル間で行を一致させることができません。

</File>

### フルリフレッシュ

ベストプラクティスとして、マイクロバッチモデルで [`full_refresh: false` を設定](/reference/resource-configs/full_refresh) し、`--full-refresh` フラグを指定した呼び出しを無視することを推奨します。履歴データを再処理する必要がある場合は、開始日と終了日を明示的に指定したターゲットバックフィルを使用してください。

## 使用方法

**モデルクエリは、正確に 1 つの「バッチ」のデータだけを処理（読み取りと返却）するように記述する必要があります**。これは、単純化のための前提であり、かつ強力な前提です。
- `is_incremental` フィルタリングについて考える必要はありません。
- DML 戦略（upsert/merge/replace）から選択する必要はありません。
- モデルをプレビューし、特定のバッチが処理されてテーブルに書き込まれたときに表示されるレコードを正確に確認できます。

マイクロバッチモデルを実行すると、dbt はロードする必要があるバッチを評価し、バッチごとに SQL クエリに分割して、それぞれを個別にロードします。

dbt は、このモデルの `lookback` および `batch_size` 設定に基づいて、`event_time` を定義する上流の入力（`source` または `ref`）を自動的にフィルタリングします。

標準の増分実行中、dbt は現在のタイムスタンプと構成された `lookback` に従ってバッチを処理し、バッチごとに 1 つのクエリを実行します。

<Lightbox src="/img/docs/building-a-dbt-project/microbatch/microbatch_lookback.png" title="Configure a lookback to reprocess additional batches during standard incremental runs"/>

**注:** 上流モデルで `event_time` を設定しているものの、その参照をフィルタリングしたくない場合は、`ref('upstream_model').render()` を指定して自動フィルタリングを無効にできます。ただし、これは一般的には推奨されません。`event_time` を設定するモデルの多くはかなり大きく、参照がフィルタリングされていない場合、各バッチでこの入力テーブル全体をスキャンしてしまうためです。

## バックフィル

エラーのあるソースデータを修正する場合でも、ビジネスロジックの変更を遡及的に適用する場合でも、大量の履歴データを再処理する必要がある場合があります。

マイクロバッチモデルのバックフィルは、実行またはビルドを選択し、`event_time` に「開始」と「終了」を指定するだけです。`--event-time-start` と `--event-time-end` は相互に必要であり、一方を指定した場合はもう一方も必ず指定する必要があることに注意してください。

dbt は、通常どおり、開始から終了までの間のバッチを独立したクエリとして処理します。

```bash
dbt run --event-time-start "2024-09-01" --event-time-end "2024-09-04"
```


<Lightbox src="/img/docs/building-a-dbt-project/microbatch/microbatch_backfill.png" title="Configure a lookback to reprocess additional batches during standard incremental runs"/>

## 再試行

1 つ以上のバッチが失敗した場合、`dbt retry` を使用して、失敗したバッチのみを再処理できます。

![Partial retry](https://github.com/user-attachments/assets/f94c4797-dcc7-4875-9623-639f70c97b8f)

## タイムゾーン

現時点では、dbt は指定されたすべての値が UTC であると想定しています。

- `event_time`
- `begin`
- `--event-time-start`
- `--event-time-end`

将来的にはカスタムタイムゾーンのサポートを追加することも検討していますが、これらの値を UTC で定義することで、皆様の作業が楽になると考えています。

## マイクロバッチと他の増分戦略の比較

データウェアハウスがデータパーティションの同時置換/更新のための新しい操作を導入するにつれ、データウェアハウスの新しい操作が、アダプタがマイクロバッチで使用する操作よりも効率的であることが判明する場合があります。このような場合、マイクロバッチパラダイムに適合するモデルにおいて意図されたとおりに/ドキュメント化されたとおりに動作する限り、マイクロバッチのデフォルト操作を更新する権利を留保します。

ほとんどの増分モデルでは、エンドユーザー（お客様）が `{% if is_incremental() %}` 条件ブロックにフィルターを記述することで、各モデルのコンテキストにおける「新しい」意味を dbt に明示的に伝える必要があります。[`{{ this }}`](/reference/dbt-jinja-functions/this) をクエリして最新のレコードが最後にロードされた日時を確認し、オプションで遅れて到着したレコードのルックバックウィンドウを指定するような SQL を作成するのはお客様の責任です。

他の増分戦略では、テーブルへのデータの追加方法（追加のみの `insert`、`delete` + `insert`、`merge`、`insert overwrite` など）を制御しますが、これらにはすべて共通点があります。

例:

```sql
{{
    config(
        materialized='incremental',
        incremental_strategy='delete+insert',
        unique_key='date_day'
    )
}}

select * from {{ ref('stg_events') }}

    {% if is_incremental() %}
        -- this filter will only be applied on an incremental run
        -- add a lookback window of 3 days to account for late-arriving records
        where date_day >= (select {{ dbt.dateadd("day", -3, "max(date_day)") }} from {{ this }})  
    {% endif %}

```

この増分モデルでは、以下のようになります:

- 「新しい」レコードとは、`date_day` が、以前に読み込まれた最大の `date_day` よりも大きいレコードです。
- ルックバックウィンドウは 3 日間です。
- 特定の `date_day` に新しいレコードがある場合、`date_day` の既存のデータは削除され、新しいデータが挿入されます。

先ほどと同じ例で、新しい `microbatch` 増分戦略を使用してみましょう:

<File name="models/staging/stg_events.sql">

```sql
{{
    config(
        materialized='incremental',
        incremental_strategy='microbatch',
        event_time='event_occured_at',
        batch_size='day',
        lookback=3,
        begin='2020-01-01',
        full_refresh=false
    )
}}

select * from {{ ref('stg_events') }} -- this ref will be auto-filtered
```

</File>

モデルの直接の親（この場合は `stg_events`）にも `event_time` を設定しています:

<File name="models/staging/stg_events.yml">

```yaml
models:
  - name: stg_events
    config:
      event_time: my_time_field
```

</File>

これで完了です！

モデルを実行すると、各バッチで個別のクエリがテンプレート化されます。例えば、10月1日にモデルを実行した場合、dbtは9月28日から10月1日までの各日について個別のクエリをテンプレート化します（合計4つのバッチ）。

`2024-10-01` のクエリは次のようになります:

<File name="target/compiled/staging/stg_events.sql">

```sql
select * from (
    select * from "analytics"."stg_events"
    where my_time_field >= '2024-10-01 00:00:00'
      and my_time_field < '2024-10-02 00:00:00'
)
```

</File>

データ プラットフォームに基づいて、dbt は最も効率的なアトミック メカニズムを選択して、既存のテーブルにこれら 4 つのバッチ (`2024-09-28`、`2024-09-29`、`2024-09-30`、および `2024-10-01`) を挿入、更新、または置換します。
