---
title: "セマンティックモデル"
id: "semantic-models"
description: "Semantic models are yml abstractions on top of a dbt mode, connected via joining keys as edges"
keywords:
  - dbt metrics layer
sidebar_label: Semantic models
tags: [Metrics, Semantic Layer]
pagination_next: "docs/build/dimensions"
---

import CopilotBeta from '/snippets/_dbt-copilot-avail.md';

<CopilotBeta resource='semantic models' />

セマンティックモデルは、MetricFlow におけるデータ定義の基盤であり、dbt セマンティックレイヤーの基盤となります。

- セマンティックモデルは、セマンティックグラフ内のエンティティによって接続されたノードと考えてください。
- MetricFlow は、メトリクスのクエリを実行するために、YAML 構成ファイルを使用してこのグラフを作成します。
- 各セマンティックモデルは DAG 内の dbt モデルに対応しており、各セマンティックモデルには固有の YAML 構成が必要です。
- 各セマンティックモデルに一意の名前を付ければ、1 つの dbt モデル（SQL または Python）から複数のセマンティックモデルを作成できます。
- セマンティックモデルは、dbt プロジェクトディレクトリ内の YAML ファイルで構成します。プロジェクト構造の詳細については、[ベストプラクティスガイド](/best-practices/how-we-build-our-metrics/semantic-layer-1-intro) を参照してください。
- 必要に応じて、`metrics:` フォルダ内またはプロジェクトソース内に整理します。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/semantic_foundation.jpg" width="70%" title="A semantic model is made up of different components: Entities, Measures, and Dimensions."/>

import SLCourses from '/snippets/\_sl-course.md';

<SLCourses/>

ここでは、セマンティック モデルのコンポーネントを例とともに説明します:

| Component    | Description      | Required     |  Type     | 
| ------------ | ---------------- | -------- | -------- | 
| [Name](#name)     | セマンティックモデルには一意の名前を選択してください。名前に二重アンダースコア (\_\_) を使用することはサポートされていないため、避けてください。   | Required | String |
| [Description](#description)    | 	説明に重要な詳細が含まれています。   | Optional | String |
| [Model](#model)     | `ref` 関数を使用して、セマンティック モデルの dbt モデルを指定します。 | Required | String |
| [Defaults](#defaults)      | モデルのデフォルトでは、現在は `agg_time_dimension` のみがサポートされています。   | Required |  Dict |
| [Entities](#entities)         | エンティティの列を結合キーとして使用し、`type` パラメータを使用してそのタイプを主キー、外部キー、または一意キーとして指定します。  | Required | List | 
| [Primary Entity](#primary-entity) | プライマリエンティティが存在する場合、このコンポーネントはオプションです。セマンティックモデルにプライマリエンティティが存在しない場合は、このプロパティは必須です。 | Optional | String | 
| [Dimensions](#dimensions)     | メトリックのデータをグループ化またはスライスするさまざまな方法。`time` または `categorical` になります。 | Required | List |
| [Measures](#measures)     | データモデルの列に適用される集計。最終的な指標として使用することも、より複雑な指標の構成要素として使用することもできます。  | Optional | List |
| [Label](#label)     | セマンティック モデルの `node`、`dimension`、`entity`、および/または `measures` の表示名。  | Optional | String |
| `config`   | メトリックの設定を指定するには、[`config`](/reference/resource-properties/config) プロパティを使用します。[`meta`](/reference/resource-configs/meta)、[`group`](/reference/resource-configs/group)、[`enabled`](/reference/resource-configs/enabled) 設定をサポートしています。 | Optional | Dict |

## セマンティックモデルのコンポーネント

セマンティックモデルの完全な仕様は以下の通りです:

```yaml
semantic_models:
  - name: the_name_of_the_semantic_model ## Required
    description: same as always ## Optional
    model: ref('some_model') ## Required
    defaults: ## Required
      agg_time_dimension: dimension_name ## Required if the model contains measures
    entities: ## Required
      - see more information in entities
    measures: ## Optional
      - see more information in the measures section
    dimensions: ## Required
      - see more information in the dimensions section
    primary_entity: >-
      if the semantic model has no primary entity, then this property is required. #Optional if a primary entity exists, otherwise Required
```

プロジェクト構造の詳細については、[ベストプラクティスガイド](/best-practices/how-we-build-our-metrics/semantic-layer-1-intro)を参照してください。

以下の例は、完全な構成と各フィールドの詳細な説明を示しています:

```yaml
semantic_models:
  - name: transaction # A semantic model with the name Transactions
    model: ref('fact_transactions') # References the dbt model named `fact_transactions`
    description: "Transaction fact table at the transaction level. This table contains one row per transaction and includes the transaction timestamp."
    defaults:
      agg_time_dimension: transaction_date

    entities: # Entities included in the table are defined here. MetricFlow will use these columns as join keys.
      - name: transaction
        type: primary
        expr: transaction_id
      - name: customer
        type: foreign
        expr: customer_id

    dimensions: # dimensions are qualitative values such as names, dates, or geographical data. They provide context to metrics and allow "metric by group" data slicing.
      - name: transaction_date
        type: time
        type_params:
          time_granularity: day

      - name: transaction_location
        type: categorical
        expr: order_country

    measures: # Measures are columns we perform an aggregation over. Measures are inputs to metrics.
      - name: transaction_total
        description: "The total value of the transaction."
        agg: sum

      - name: sales
        description: "The total sale of the transaction."
        agg: sum
        expr: transaction_total

      - name: median_sales
        description: "The median sale of the transaction."
        agg: median
        expr: transaction_total

  - name: customers # Another semantic model called customers.
    model: ref('dim_customers')
    description: "A customer dimension table."

    entities:
      - name: customer
        type: primary
        expr: customer_id

    dimensions:
      - name: first_name
        type: categorical
```

セマンティック モデルは、スキーマ ファイルまたはプロジェクト レベルで [`meta`](/reference/resource-configs/meta)、[`group`](/reference/resource-configs/group)、および [`enabled`](/reference/resource-configs/enabled) [`config`](/reference/resource-properties/config) プロパティをサポートします:

- `models/semantic.yml` 内のセマンティック モデル構成:

  ```yml
  semantic_models:
    - name: orders
      config:
        enabled: true | false
        group: some_group
        meta:
          some_key: some_value
  ```

- `dbt_project.yml` 内のセマンティック モデル構成:

  ```yml
  semantic-models:
    my_project_name:
      +enabled: true | false
      +group: some_group
      +meta:
        some_key: some_value
  ```

`dbt_project.yml` と設定の命名規則の詳細については、[dbt_project.yml リファレンス ページ](/reference/dbt_project.yml#naming-convention) を参照してください。

### Name

セマンティックモデルの名前を定義します。
セマンティックモデルには一意の名前を定義する必要があります。
セマンティックグラフはこの名前を使用してモデルを識別します。この名前はいつでも更新できます。
名前に二重アンダースコア (\_\_) を使用しないでください。サポートされていません。

### Description

セマンティックモデルの説明に重要な詳細が含まれます。
この説明は主に他の設定コントリビューターによって使用されます。
パイプ演算子 `(|)` を使用して、説明に複数行を含めることができます。

### Model

[`ref` 関数](/reference/dbt-jinja-functions/ref)を使用して、セマンティック モデルの dbt モデルを指定します。

### Defaults

セマンティックモデルのデフォルト。
現在は `agg_time_dimension` のみです。
`agg_time_dimension` は、メジャーのデフォルトの時間ディメンションを表します。
これは、メジャーに `agg_time_dimension` キーを直接追加することで上書きできます。
例については [Dimensions](/docs/build/dimensions) を参照してください。

### Entities

モデル内の [エンティティ](/docs/build/entities) を指定するには、列を結合キーとして使用し、type パラメータを使用して、その `type` を主キー、外部キー、または一意キーとして指定します。

### Primary entity

MetricFlow では、すべてのディメンションをエンティティに関連付ける必要があります。
これは、ディメンション名の一意性を保証するためです。
データソースにプライマリエンティティがない場合は、`primary_entity: entity_name` キーを使用してエンティティに名前を割り当てる必要があります。
必ずしもそのテーブル内の列にマッピングする必要はなく、名前を割り当ててもクエリ生成には影響しません。

プライマリエンティティは、次の設定を使用して定義できます:

```yaml
semantic_model:
  name: bookings_monthly_source
  description: bookings_monthly_source
  defaults:
    agg_time_dimension: ds
  model: ref('bookings_monthly_source')
  measures:
    - name: bookings_monthly
      agg: sum
      create_metric: true
  primary_entity: booking_id
```

<Tabs>

<TabItem value="entitytypes" value="Entity types">

キーの種類は次のとおりです。

- **主キー** - テーブル内の行ごとに1つのレコードのみが存在し、データプラットフォーム内のすべてのレコードが含まれます。
- **一意キー** - テーブル内の行ごとに1つのレコードのみが存在しますが、データプラットフォーム内のレコードのサブセットが含まれる場合があります。
null値が含まれる場合もあります。
- **外部キー** - 同じレコードが0個、1個、または複数個存在する場合があります。
null値が含まれる場合もあります。
- **自然キー** - 実際のデータに基づいてレコードを一意に識別する、テーブル内の列または列の組み合わせ。
たとえば、`sales_person_id` は `sales_person_department` ディメンションテーブルで自然キーとして機能します。

</TabItem>
<TabItem value="sample" label="Sample config">

この例では、3つのエンティティとそのエンティティタイプ（`transaction`（プライマリ）、`order`（外部）、`user`（外部））を持つセマンティックモデルを示しています。

目的の列を参照するには、モデルの実際の列名を`name`パラメータに指定します。
また、`name`をエイリアスとして使用して列名を変更したり、`expr`パラメータを使用して元の列名または列のSQL式を参照したりすることもできます。

```yaml
entity:
  - name: transaction
    type: primary
  - name: order
    type: foreign
    expr: id_order
  - name: user
    type: foreign
    expr: substring(id_order FROM 2)
```

セマンティックモデル内のエンティティ（結合キー）は、`name` パラメータを使用して参照できます。
エンティティ名はセマンティックモデル内で一意である必要があります。また、識別子名は MetricFlow が [結合](/docs/build/join-logic) に使用するため、セマンティックモデル間で一意でなくても構いません。 <!--You can also create [composite keys](/docs/build/entities#composite-keys), like in event logs where a unique ID is a combination of timestamp, event type keys, and machine IDs.-->

</TabItem>
</Tabs>

### Dimensions

[ディメンション](/docs/build/dimensions) は、データを整理したり表示したりする様々な方法です。
これらは、実質的にはメトリクスのグループ化パラメータです。
たとえば、地域、国、役職などでデータをグループ化できます。

MetricFlow は、メトリクスでディメンションを利用できるようにする際、動的なアプローチを採用しています。
MetricFlow では、事前にすべてのグループ化の可能性を把握するのではなく、必要なディメンションを要求し、クエリ時に要求されたディメンションに到達するために必要な結合を構築します。
このアプローチの利点は、データのグループ化のあらゆる方法を事前に実現するシステムをセットアップする必要がないことです。これは時間がかかり、エラーが発生しやすくなります。
代わりに、セマンティックモデル内で必要なディメンション（グループ化パラメータ）を定義すると、有効なメトリクスで自動的に利用できるようになります。

ディメンションには以下の特性があります。

- ディメンションには、カテゴリディメンションと時間ディメンションの2種類があります。
カテゴリディメンションは数値で測定できないものを対象とし、時間ディメンションは日付やタイムスタンプを表します。
- ディメンションは、定義されているセマンティックモデルのプライマリエンティティにバインドされます。
例えば、プライマリエンティティとして「user」を持つモデルで「full_name」というディメンションが定義されている場合、「full_name」のスコープは「user」エンティティに限定されます。
このディメンションを参照するには、完全修飾ディメンション名「user__full_name」を使用します。
- ディメンションの名前は、同じプライマリエンティティを持つ各セマンティックモデル内で一意である必要があります。
異なるプライマリエンティティを持つセマンティックモデルで定義されている場合、ディメンション名は重複しても構いません。


:::info 時間グループの場合

メジャーを含むセマンティックモデルの場合、[プライマリ時間グループ](/docs/build/dimensions#time)が必要です。
:::

### Measures

[メジャー](/docs/build/measures) は、データモデルの列に適用される集計です。
より複雑な指標の基礎となる構成要素として使用することも、最終的な指標そのものとして使用することもできます。

メジャーには様々なパラメータがあり、その説明とタイプとともに表形式で一覧表示されます。

import MeasuresParameters from '/snippets/\_sl-measures-parameters.md';

<MeasuresParameters />

import SetUpPages from '/snippets/\_metrics-dependencies.md';

<SetUpPages />

## Related docs

- [About MetricFlow](/docs/build/about-metricflow)
- [Dimensions](/docs/build/dimensions)
- [Entities](/docs/build/entities)
- [Measures](/docs/build/measures)
- [Semantic Layer best practices guide](/best-practices/how-we-build-our-metrics/semantic-layer-1-intro)
