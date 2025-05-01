---
title: "MetricFlowについて"
id: about-metricflow
description: "MetricFlowとその主要概念について詳しく学ぶ"
sidebar_label: About MetricFlow
tags: [Metrics, Semantic Layer]
pagination_next: "docs/build/join-logic"
pagination_prev: null
---

このガイドでは、MetricFlow を初めて使用する方のために、その基本的な考え方を紹介します。dbt セマンティック レイヤーを支える MetricFlow は、企業のメトリクスのロジックの定義と管理を支援します。
MetricFlow は独自の抽象化セットであり、データ コンシューマーがデータ プラットフォームからメトリクス データセットを迅速かつ効率的に取得するのに役立ちます。

MetricFlow は SQL クエリの構築を処理し、dbt セマンティック モデルとメトリクスの仕様を定義します。
MetricFlow を使用すると、dbt プロジェクトでメトリクスを定義し、[MetricFlow コマンド](/docs/build/metricflow-commands) を使用して dbt Cloud でも dbt Core でもクエリを実行できます。

開始する前に、以下のガイドラインをご確認ください。

- YAML でメトリクスを定義し、これらの [新しいメトリクス仕様](https://github.com/dbt-labs/dbt-core/discussions/7456) を使用してクエリを実行します。
- MetricFlow を使用するには、[dbt バージョン](/docs/dbt-versions/upgrade-dbt-version-in-cloud) 1.6 以上が必要です。
- MetricFlow は、Snowflake、BigQuery、Databricks、Postgres (dbt Core のみ)、または Redshift で使用できます。
- [dbt セマンティック レイヤー](/docs/use-dbt-semantic-layer/dbt-sl) とその多様な [利用可能な統合](/docs/cloud-integrations/avail-sl-integrations) を使用して、インサイトを発見し、メトリクスをクエリします。

## MetricFlow

MetricFlow は、多様なビジネスニーズに対応するために、異なるデータディメンションにわたるメトリクス作成を効率化するように設計された SQL クエリ生成ツールです。
- YAML ファイルを介して動作し、セマンティックグラフによって言語とデータがリンクされます。このグラフは、[セマンティックモデル](/docs/build/semantic-models) (データエントリポイント) と [メトリクス](/docs/build/metrics-overview) (定量指標を作成するための関数) で構成されます。
- MetricFlow は [BSL パッケージ](https://github.com/dbt-labs/metricflow) であり、コードソースが利用可能で、dbt バージョン 1.6 以降と互換性があります。データ実践者や熱心な開発者の皆様の貢献を強く推奨します。
- dbt セマンティックレイヤーの一部である MetricFlow は、YAML 抽象化を使用して組織がメトリクスを定義できるようにします。
- メトリクスディメンション、ディメンション値のクエリ、および構成の検証を行うには、[MetricFlow コマンド](/docs/build/metricflow-commands) を使用します。


**注** &mdash; MetricFlow は現在、dbt [組み込み関数またはパッケージ](/reference/dbt-jinja-functions/builtins) をサポートしていませんが、将来的にサポートされる予定です。

MetricFlow は以下の原則に従います。

- **完全性と柔軟性**: あらゆるデータモデルで柔軟な抽象化を使用してメトリックロジックを定義します。
- **DRY (Don't Repeat Yourself)**: 可能な限りメトリック定義を有効にすることで冗長性を最小限に抑えます。
- **段階的な複雑さを伴うシンプルさ:** 使い慣れたデータモデリングの概念を使用して MetricFlow にアプローチします。
- **パフォーマンスと効率性**: 集中型データエンジニアリングと分散型ロジック所有権をサポートしながら、パフォーマンスを最適化します。

### セマンティックグラフ

「セマンティックグラフ」という新しい概念を導入します。
これは、セマンティックモデルとYAML構成の関係性によって、メトリクスを構築するためのデータランドスケープを構築します。
これは地図のようなもので、テーブルは場所、テーブル間の接続（エッジ）は道路のようなものです。
セマンティックグラフは内部的には<Term id="dag" />のサブセットであり、セマンティックモデルはDAG上のノードとして表示されます。

セマンティックグラフは、どの情報が利用可能で、どの情報が利用不可能かを判断するのに役立ちます。
セマンティックグラフにおけるテーブル間の接続は、情報間の関係性をより明確に表しています。
これは、接続がタスク間の依存関係を示すDAGとは異なります。

MetricFlowは、メトリクスを生成する際に、SQLエンジンを使用して、セマンティックモデルとメトリクス用のYAMLファイルで定義されたフレームワークに基づき、テーブル間の最適なパスを計算します。
これらのモデルとメトリックが正しく定義されると、dbt セマンティック レイヤーの統合により下流で使用できるようになります。

### セマンティックモデル

セマンティックモデルはデータの出発点であり、dbt プロジェクト内のモデルに対応します。各モデルから複数のセマンティックモデルを作成できます。セマンティックモデルには、データテーブルのようなメタデータがあり、グラフを正しくナビゲートするために必要なテーブル名や主キーなどの重要な情報が定義されています。

セマンティックモデルには、主に 3 つのメタデータがあります。

* [エンティティ](/docs/build/entities) - セマンティックモデルの結合キー（セマンティックモデル間のトラバーサルパス、またはエッジと考えてください）。
* [ディメンション](/docs/build/dimensions) - メトリクスをグループ化または細分化する方法。
* [メジャー](/docs/build/measures) - 数値結果を返す集計関数で、メトリクスの作成に使用できます。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/semantic_foundation.jpg" width="70%" title="A semantic model is made up of different components: Entities, Measures, and Dimensions."/>

### メトリクス

重要な概念であるメトリクスは、メジャー、制約、またはその他の数学関数を組み合わせて新しい定量的指標を定義する関数です。MetricFlow は、メジャーと、平均、合計、個別カウントなどのさまざまな集計タイプを使用してメトリクスを作成します。ディメンションはメトリクスにコンテキストを追加します。ディメンションがなければ、メトリクスは単なる数値になってしまいます。メトリクスは、セマンティックモデルと同じ YAML ファイルで定義することも、新しいファイルを作成することもできます。

MetricFlow は、さまざまなメトリクスタイプをサポートしています。

- [コンバージョン](/docs/build/conversion) - 設定された期間内に、エンティティのベースイベントとそれに続くコンバージョンイベントが発生したタイミングを追跡するのに役立ちます。
- [累積](/docs/build/cumulative) - 指定された期間のメジャーを集計します。
- [派生](/docs/build/derived) - 他のメトリクスの式で、メトリクスに基づいて計算を行うことができます。
- [比率](/docs/build/ratio) - 顧客あたりの収益など、2 つの指標から比率を作成します。
- [シンプル](/docs/build/simple) - 1 つの指標を直接参照する指標。

## ユースケース

次のセクションでは、データ実務者が現在どのようにメトリクスを計算しているかを示し、MetricFlow がどのようにメトリクスの定義を容易かつ柔軟にするかを比較します。

以下のサンプルデータは、Jaffle Shop リポジトリに基づいています。[dbt プロジェクト](https://github.com/dbt-labs/jaffle-sl-template) の全文をご覧いただけます。サンプルモデルで使用しているテーブルは次のとおりです。

- `orders` は、分析用にクリーンアップおよび整理された本番環境データプラットフォームのエクスポートです。
- `customers` は、この場合は部分的に非正規化されたテーブルで、上流のプロセスを通じて order テーブルから派生した列が含まれています。

<!-- ![MetricFlow-SchemaExample](/img/docs/building-a-dbt-project/MetricFlow-SchemaExample.jpeg) -->

これをより具体的にするために、SQL 式を使用して定義される指標「order_total」を考えてみましょう。

「select sum(order_total) as order_total from orders」
この式は、orders テーブルの order_total 列を合計することで、すべての注文の合計収益を計算します。ビジネス環境では、指標「order_total」は、次のようなさまざまなカテゴリに従って計算されることがよくあります。
- 時間（例：date_trunc(ordered_at, 'day')）
- 注文タイプ（「orders」テーブルの「is_food_order」ディメンションを使用）

### メトリクスの計算

次に、データ実務者が現在複数のクエリを使ってメトリクスを計算する方法と、MetricFlow がどのようにプロセスを簡素化・効率化しているかを比較します。

<Tabs>
<TabItem value="mulqueries" label="Calculate with multiple queries">

以下の例は、データ担当者が一般的に「order_total」指標の集計値を計算する方法を示しています。アナリストは、新規顧客からの収益額など、指標に関するより詳細な情報を求められることも少なくありません。

以下のクエリを使用すると、複数のアナリストがそれぞれ独自のクエリ手法を用いて同じデータを処理する状況が発生します。これは混乱や不整合を引き起こし、データ管理の煩雑さにつながる可能性があります。

```sql
select
    date_trunc('day',orders.ordered_at) as day, 
    case when customers.first_ordered_at is not null then true else false end as is_new_customer,
    sum(orders.order_total) as order_total
from
  orders
left join
  customers
on
  orders.customer_id = customers.customer_id
group by 1, 2
```

</TabItem>
<TabItem value="metricflow" label="Calculate with MetricFlow">

次の 3 つのサンプル タブでは、MetricFlow を使用して、order_total をメトリックとして使用するセマンティック モデルとサンプル スキーマを定義し、一貫性のある正確な結果を作成します。これにより、混乱やコードの重複がなくなり、ワークフローが合理化されます。

<Tabs>
<TabItem value="example1" label="Revenue example">

この例では、「orders」テーブルの「order_total」列に基づいて、「order_total」というメジャーが定義されています。

時間ディメンション「metric_time」は日単位の粒度を提供し、週単位または月単位の期間に集計できます。さらに、「customers」セマンティックモデルには、「is_new_customer」というカテゴリディメンションが指定されています。


```yaml
semantic_models:
  - name: orders    # The name of the semantic model
    description: |
      A model containing order data. The grain of the table is the order id.
    model: ref('orders') #The name of the dbt model and schema
    defaults:
      agg_time_dimension: metric_time
    entities: # Entities, which usually correspond to keys in the table. 
      - name: order_id
        type: primary
      - name: customer
        type: foreign
        expr: customer_id
    measures:   # Measures, which are the aggregations on the columns in the table.
      - name: order_total
        agg: sum
    dimensions: # Dimensions are either categorical or time. They add additional context to metrics and the typical querying pattern is Metric by Dimension.
      - name: metric_time
        expr: cast(ordered_at as date)
        type: time
        type_params:
          time_granularity: day
  - name: customers    # The name of the second semantic model
    description: >
      Customer dimension table. The grain of the table is one row per
        customer.
    model: ref('customers') #The name of the dbt model and schema
    defaults:
      agg_time_dimension: first_ordered_at
    entities: # Entities, which  usually correspond to keys in the table.
      - name: customer 
        type: primary
        expr: customer_id
    dimensions: # Dimensions are either categorical or time. They add additional context to metrics and the typical querying pattern is Metric by Dimension.
      - name: is_new_customer
        type: categorical
        expr: case when first_ordered_at is not null then true else false end
      - name: first_ordered_at
        type: time
        type_params:
          time_granularity: day

  ```

</TabItem>
<TabItem value="example2" label="More dimensions example">

同様に、セマンティック モデルに `is_food_order` などのディメンションを追加して、さらに多くのディメンションを組み込んで、収益の order_total を細分化することもできます。

```yaml
semantic_models:
  - name: orders
    description: |
      A model containing order data. The grain of the table is the order id.
    model: ref('orders')  #The name of the dbt model and schema
    defaults:
      agg_time_dimension: metric_time
    entities: # Entities, which usually correspond to keys in the table
      - name: order_id
        type: primary
      - name: customer
        type: foreign
        expr: customer_id
    measures: # Measures, which are the aggregations on the columns in the table.
      - name: order_total
        agg: sum
    dimensions: # Dimensions are either categorical or time. They add additional context to metrics and the typical querying pattern is Metric by Dimension.
      - name: metric_time
        expr: cast(ordered_at as date)
        type: time
        type_params:
          time_granularity: day
      - name: is_food_order
        type: categorical
```
</TabItem>
<TabItem value="example3" label="Advanced example">

リピーターからの毎日の食品注文による収益額など、さらに複雑な指標が必要だと想像してみてください。MetricFlowがなければ、データ担当者が最初に書くSQLは次のようになるでしょう。

```sql
select
    date_trunc('day',orders.ordered_at) as day, 
    sum(case when is_food_order = true then order_total else null end) as food_order,
    sum(orders.order_total) as sum_order_total,
    food_order/sum_order_total
from
  orders
left join
  customers
on
  orders.customer_id = customers.customer_id
where
  case when customers.first_ordered_at is not null then true else false end = true
group by 1
```

MetricFlowは、以下に示すように、メトリックYAML設定を介してSQLプロセスを簡素化します。また、これらの設定をGitリポジトリにコミットすることで、データチームとビジネスチームの全員が、真に唯一の情報源として確認・承認できるようになります。

```yaml
metrics:
  - name: food_order_pct_of_order_total_returning
    description: Revenue from food orders from returning customers
    label: "Food % of Order Total"
    type: ratio
    type_params:
      numerator: food_order
      denominator: order_total
    filter: |
      {{ Dimension('customer__is_new_customer') }} = false
```
</TabItem>
</Tabs>

</TabItem>
</Tabs>

## FAQs

<DetailsToggle alt_header="データセットを正規化する必要がありますか?">

いいえ、全く問題ありません！クリーンアップされ、適切にモデル化されたデータセットは非常に強力で理想的な入力データですが、生のデータセットから完全に非正規化されたデータセットまで、あらゆるデータセットを使用できます。

上流アプリケーションでは、不良データのフィルタリング、共通オブジェクトの正規化、キーとテーブルのデータモデリングなど、高品質なデータ整合性を適用することをお勧めします。セマンティックレイヤーは、正規化ではなく非正規化を行う方が効率的です。

データ整合性に投資していなくても問題ありません。セマンティックレイヤーは、SQLクエリや式を使用して一貫性のあるデータセットを定義できます。

</DetailsToggle>

<DetailsToggle alt_header="正規化されたデータが理想的な入力であるのはなぜですか?">

MetricFlowは、非正規化を効率的に行うために構築されています。生のデータセットを取得し、データの一貫性と整理されたデータモデルの構築に必要な様々なタスクを実行するための、より優れたツールは存在します。一方で、非正規化されたデータを入力すると、技術的に管理が困難な冗長性が生じる可能性があり、MetricFlowがメトリクスを集約するために使用できる粒度が低下する可能性があります。
</DetailsToggle>

<DetailsToggle alt_header="指標を測定基準と同じにするのはなぜですか?">
MetricFlow の原則の一つは、ロジックの重複を減らすことです。これは Don't Repeat Yourself (DRY) とも呼ばれます。

多くのメトリクスは再利用されたメジャーから構築され、場合によっては異なるセマンティックモデルのメジャーから構築されます。これにより、メトリクスは深さ優先（複数のメトリクスが互いの関数として機能する）ではなく、幅優先（単独で機能するメトリクス）で構築できます。

さらに、すべてのメトリクスがメジャーに基づいて構築されるわけではありません。例えば、コンバージョンメトリクスは、あるイベントレコードの後に​​別のイベントレコードが存在するかどうかとして定義される可能性があります。

</DetailsToggle>
<DetailsToggle alt_header="dbt セマンティック レイヤーは結合をどのように処理しますか?">
MetricFlow を搭載した dbt セマンティックレイヤーは、エンティティに渡されるキーとパラメータの型に基づいて結合を構築します。結合の構築方法について詳しくは、結合の種類に関するドキュメントをご覧ください。

MetricFlow は、任意の結合ロジックをキャプチャするのではなく、各識別子の型をキャプチャし、ユーザーが適切な結合を選択できるように支援します。これにより、ファンアウト結合やキャズム結合の構築を回避し、読みやすい SQL を生成できます。
</DetailsToggle>

<DetailsToggle alt_header="エンティティと結合キーは同じものですか?">
エンティティを結合キーとして考えると分かりやすいでしょう。MetricFlow のエンティティは、2 つのテーブルを結合する以外にも、ディメンションとして機能するなど、さまざまな用途に使用できます。
</DetailsToggle>

<DetailsToggle alt_header="プライマリ エンティティまたは一意のエンティティのないテーブルにディメンションを設定できますか?">
はい、可能です。ただし、ディメンションはテーブルの主要素または一意の要素の属性とみなされるため、そのテーブルで定義された指標でのみ使用できます。他のテーブルの指標と結合することはできません。これはイベントログでよく見られる現象です。
</DetailsToggle>

## Related docs
- [Joins](/docs/build/join-logic)
- [Validations](/docs/build/validation)
