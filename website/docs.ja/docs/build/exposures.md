---
title: "DAGにエクスポージャーを追加する"
sidebar_label: "Exposures"
id: "exposures"
---

エクスポージャーを使用すると、ダッシュボード、アプリケーション、データサイエンス パイプラインなど、dbt プロジェクトの下流での使用を定義および記述できます。エクスポージャーを定義することで、次のことが可能になります。
- エクスポージャーにフィードされるリソースを実行、テスト、および一覧表示する
- 自動生成された [ドキュメント](/docs/build/documentation) サイトに、データ コンシューマーに関連するコンテキストを専用ページとして表示する

エクスポージャーは、以下の 2 つの方法で定義できます。
- 手動 - プロジェクトの YAML ファイルで [明示的に](/docs/build/exposures#declaring-an-exposure) 宣言します。
- 自動 - dbt Cloud は、サポートされている統合に対して [ダウンストリーム エクスポージャーを自動的に作成および視覚化](/docs/cloud-integrations/downstream-exposures) するため、手動での YAML 定義は不要になります。
これらのダウンストリーム エクスポージャーは、dbt のメタデータ システムに保存され、[dbt Explorer](/docs/collaborate/explore-projects) に表示され、手動で定義したエクスポージャーと同様に動作します。
ただし、YAML ファイルには存在しません。

### エクスポージャーの宣言

エクスポージャーは、`exposures:` キーの下にネストされた `.yml` ファイルで定義されます。

次の例は、`models/<filename>.yml` ファイル内のエクスポージャー定義を示しています:

<File name='models/<filename>.yml'>

```yaml
version: 2

exposures:

  - name: weekly_jaffle_metrics
    label: Jaffles by the Week
    type: dashboard
    maturity: high
    url: https://bi.tool/dashboards/1
    description: >
      Did someone say "exponential growth"?

    depends_on:
      - ref('fct_orders')
      - ref('dim_customers')
      - source('gsheets', 'goals')
      - metric('count_orders')

    owner:
      name: Callum McData
      email: data@jaffleshop.com
```

</File>

### 使用可能なプロパティ

_必須:_
- **name**: [スネークケース](https://en.wikipedia.org/wiki/Snake_case)で記述された一意のエクスポージャー名
- **type**: `dashboard`、`notebook`、`analysis`、`ml`、`application` のいずれか（ドキュメントサイトでの整理に使用）
- **owner**: `name` または `email` が必須。追加のプロパティも指定可能

_推奨:_
- **depends_on**: `metric`、`ref`、`source` を含む参照可能なノードのリスト。`exposure` が `source` に直接依存することは可能ですが、ほとんどありません。

_省略可能:_
- **label**: スペース、大文字、または特殊文字を含めることができます。
- **url**: 生成されたドキュメントサイトの右上隅にある「**このエクスポージャーを表示**」へのリンクを有効化して表示します。
- **maturity**: エクスポージャーの信頼性または安定性のレベルを示します。「高」、「中」、「低」のいずれかになります。
例えば、組織内で広く使用され、信頼されている、確立されたダッシュボードには「高」成熟度を使用できます。
新しい分析や実験的な分析には「低」成熟度を使用します。

_General properties (optional)_

- [**description**](/reference/resource-properties/description)
- [**tags**](/reference/resource-configs/tags)
- [**meta**](/reference/resource-configs/meta)
- [**enabled**](/reference/resource-configs/enabled) &mdash; このプロパティは、[`dbt_project.yml`](/reference/dbt_project.yml) ファイル内の公開レベルまたはプロジェクト レベルで設定できます。

### エクスポージャーの参照

エクスポージャーを定義したら、それを参照するコマンドを実行できます：

```
dbt run -s +exposure:weekly_jaffle_report
dbt test -s +exposure:weekly_jaffle_report

```

[dbt Explorer サイト](/docs/collaborate/explore-projects) を生成すると、公開内容が表示されます:

<Lightbox src="/img/docs/building-a-dbt-project/dbt-explorer-exposures.jpg" title="Exposures has a dedicated section, under the 'Resources' tab in dbt Explorer,  which lists each exposure in your project."/>
<Lightbox src="/img/docs/building-a-dbt-project/dag-exposures.png" title="Exposures appear as nodes in the dbt Explorer DAG. It displays an orange 'EXP' indicator within the node. "/>

## 関連ドキュメント

* [エクスポージャープロパティ](/reference/exposure-properties)
* [`exposure:` 選択方法](/reference/node-selection/methods#exposure)
* [データヘルスタイル](/docs/collaborate/data-tile)
