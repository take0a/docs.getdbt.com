---
title: Dimensions
id: dimensions
description: "Dimensions determine the level of aggregation for a metric, and are non-aggregatable expressions."
sidebar_label: "Dimensions"
tags: [Metrics, Semantic Layer]
---

ディメンションは、データセット内の集計不可能な列を表します。ディメンションは、データを説明または分類する属性、特徴、または特性です。
dbt セマンティック レイヤーのコンテキストでは、ディメンションはセマンティック モデルと呼ばれるより大きな構造の一部です。
ディメンションは、[エンティティ](/docs/build/entities) や [メジャー](/docs/build/measures) などの他の要素とともに作成され、データに詳細情報を追加するために使用されます。
SQL では、ディメンションは通常、SQL クエリの `group by` 句に含まれます。

<!--dimensions are non-aggregatable expressions that define the level of aggregation for a metric used to define how data is sliced or grouped in a metric. Since groups can't be aggregated, they're considered to be a property of the primary or unique entity of the table.

Groups are defined within semantic models, alongside entities and measures, and correspond to non-aggregatable columns in your dbt model that provides categorical or time-based context. In SQL, dimensions  is typically included in the GROUP BY clause.-->

すべてのディメンションには「name」と「type」が必要で、オプションで「expr」パラメータを含めることができます。
ディメンションの「name」は、同じセマンティックモデル内で一意である必要があります。

| Parameter | Description | Required | Type |
| --------- | ----------- | ---- | ---- |
| `name` |  下流ツールでユーザーに表示されるグループ名を指します。列名またはSQLクエリ参照が異なり、`expr`パラメータで指定されている場合は、エイリアスとしても機能します。<br /><br /> ディメンション名はセマンティックモデル内で一意である必要がありますが、MetricFlowは[結合](/docs/build/join-logic)を使用して適切なディメンションを識別するため、異なるモデル間では一意でない場合があります。 | Required | String |
| `type` | セマンティックモデルで作成されるグループのタイプを指定します。次の2つのタイプがあります。<br /><br />- **カテゴリ**: 地理や販売地域などの属性や特徴を記述します。<br />- **時間**: タイムスタンプや日付などの時間ベースのディメンション。 | Required | String |  
| `type_params` | 時間がプライマリであるか、パーティションとして使用されているかなどの特定のタイプのパラメータ。 | Required | Dict |
| `description` | ディメンションの明確な説明。 | Optional | String |  
| `expr` | ディメンションの基になる列またはSQLクエリを定義します。`expr` が指定されていない場合、MetricFlowはグループと同じ名前の列を使用します。列名自体を使用してSQL式を入力できます。 | Optional | String |
| `label` | 下流ツールでの表示値を定義します。プレーンテキスト、スペース、引用符（例：`orders_total` または `"orders_total"`）が使用できます。 | Optional | String |
| [`meta`](/reference/resource-configs/meta) |  リソースのメタデータを設定し、リソースを整理します。プレーンテキスト、スペース、引用符が使用できます。 | Optional | Dictionary | 

ディメンジョンの完全な仕様については以下を参照してください。

```yaml
dimensions:
  - name: Name of the group that will be visible to the user in downstream tools # Required
    type: Categorical or Time # Required
    label: Recommended adding a string that defines the display value in downstream tools. # Optional
    type_params: Specific type params such as if the time is primary or used as a partition # Required
    description: Same as always # Optional
    expr: The column name or expression. If not provided the default is the dimension name # Optional
```

セマンティック モデルでディメンションがどのように使用されるかについては、次の例を参照してください:

<VersionBlock firstVersion="1.9">

```yaml
semantic_models:
  - name: transactions
    description: A record for every transaction that takes place. Carts are considered multiple transactions for each SKU. 
    model: {{ ref('fact_transactions') }}
    defaults:
      agg_time_dimension: order_date
# --- entities --- 
  entities: 
    - name: transaction
      type: primary
      ...
# --- measures --- 
  measures: 
      ... 
# --- dimensions ---
  dimensions:
    - name: order_date
      type: time
      type_params:
        time_granularity: day
      label: "Date of transaction" # Recommend adding a label to provide more context to users consuming the data
      config: 
        meta:
          data_owner: "Finance team"
      expr: ts
    - name: is_bulk
      type: categorical
      expr: case when quantity > 10 then true else false end
    - name: type
      type: categorical
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
semantic_models:
  - name: transactions
    description: A record for every transaction that takes place. Carts are considered multiple transactions for each SKU. 
    model: {{ ref('fact_transactions') }}
    defaults:
      agg_time_dimension: order_date
# --- entities --- 
  entities: 
    - name: transaction
      type: primary
      ...
# --- measures --- 
  measures: 
      ... 
# --- dimensions ---
  dimensions:
    - name: order_date
      type: time
      type_params:
        time_granularity: day
      label: "Date of transaction" # Recommend adding a label to provide more context to users consuming the data
      expr: ts
    - name: is_bulk
      type: categorical
      expr: case when quantity > 10 then true else false end
    - name: type
      type: categorical
```
</VersionBlock>

ディメンションは、それが定義されているセマンティックモデルのプライマリエンティティにバインドされます。
例えば、ディメンション「type」は、「transaction」をプライマリエンティティとして持つモデルで定義されています。
「type」は「transaction」エンティティにスコープ指定されており、このディメンションを参照するには、完全修飾ディメンション名（例：「transaction__type」）を使用します。

MetricFlow では、すべてのセマンティックモデルにプライマリエンティティが必要です。
これは、ディメンション名が一意であることを保証するためです。
データソースにプライマリエンティティがない場合は、「primary_entity」キーを使用してエンティティに名前を割り当てる必要があります。
エンティティは必ずしもそのテーブルの列にマッピングする必要はなく、名前を割り当ててもクエリ生成には影響しません。これらの「仮想プライマリエンティティ」は、セマンティックモデル全体で一意にすることをお勧めします。プライマリエンティティ列を持たないデータソースにプライマリエンティティを定義する例を以下に示します:

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

## Dimensions types
このセクションでは、ディメンションの定義について、例を挙げながら詳しく説明します。
ディメンションには以下の種類があります:

- [Dimensions types](#dimensions-types)
- [Categorical](#categorical)
- [Time](#time)
  - [SCD Type II](#scd-type-ii)
    - [Basic structure](#basic-structure)
    - [Semantic model parameters and keys](#semantic-model-parameters-and-keys)
    - [Implementation](#implementation)
    - [SCD examples](#scd-examples)

## Categorical

カテゴリディメンションは、製品タイプなどの異なる属性、機能、特性ごとに指標をグループ化するために使用されます。
dbtモデル内の既存の列を参照したり、`expr`パラメータを指定したSQL式を使用して計算したりできます。
カテゴリディメンションの例としては、`is_bulk_transaction`があります。これは、基になる列`quantity`にcaseステートメントを適用することで作成されたグループです。
これにより、ユーザーは一括取引に基づいてデータをグループ化またはフィルタリングできます。

<VersionBlock firstVersion="1.9">

```yaml
dimensions: 
  - name: is_bulk_transaction
    type: categorical
    expr: case when quantity > 10 then true else false end
    config:
      meta:
        usage: "Filter to identify bulk transactions, like where quantity > 10."
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
dimensions: 
  - name: is_bulk_transaction
    type: categorical
    expr: case when quantity > 10 then true else false end
```
</VersionBlock>

## Time

時間には、`type_params` セクションで指定される追加パラメータがあります。
1 つ以上のメトリクスをクエリする場合、各メトリクスのデフォルトの時間ディメンションは集計時間ディメンションです。これは `metric_time` として参照するか、ディメンション名を使用できます。

複数の時間グループを別々のメトリクスで使用できます。
たとえば、`users_created` メトリクスは `created_at` を使用し、`users_deleted` メトリクスは `deleted_at` を使用します。

```bash
# dbt Cloud users
dbt sl query --metrics users_created,users_deleted --group-by metric_time__year --order-by metric_time__year

# dbt Core users
mf query --metrics users_created,users_deleted --group-by metric_time__year --order-by metric_time__year
```

You can set `is_partition` for time to define specific time spans. Additionally, use the `type_params` section to set `time_granularity` to adjust aggregation details (daily, weekly, and so on).

<Tabs queryString="dimension">

<TabItem value="is_partition" label="is_partition">

特定の期間にディメンションが存在することを示すには、`is_partition: True` を使用します。

たとえば、日付でパーティション分割されたディメンションテーブルなどです。
異なるテーブルからメトリクスをクエリする場合、dbt セマンティックレイヤーはこのパラメータを使用して、正しいディメンション値がメジャーに結合されていることを確認します。

<VersionBlock firstVersion="1.9">

```yaml
dimensions: 
  - name: created_at
    type: time
    label: "Date of creation"
    expr: ts_created # ts_created is the underlying column name from the table 
    config:
      meta:
        notes: "Only valid for orders from 2022 onward"
    is_partition: True
    type_params:
      time_granularity: day
  - name: deleted_at
    type: time
    label: "Date of deletion"
    expr: ts_deleted # ts_deleted is the underlying column name from the table
    is_partition: True 
    type_params:
      time_granularity: day

measures:
  - name: users_deleted
    expr: 1
    agg: sum
    agg_time_dimension: deleted_at
  - name: users_created
    expr: 1
    agg: sum
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
dimensions: 
  - name: created_at
    type: time
    label: "Date of creation"
    expr: ts_created # ts_created is the underlying column name from the table 
    is_partition: True
    type_params:
      time_granularity: day
  - name: deleted_at
    type: time
    label: "Date of deletion"
    expr: ts_deleted # ts_deleted is the underlying column name from the table
    is_partition: True 
    type_params:
      time_granularity: day

measures:
  - name: users_deleted
    expr: 1
    agg: sum
    agg_time_dimension: deleted_at
  - name: users_created
    expr: 1
    agg: sum
```
</VersionBlock>

</TabItem>

<TabItem value="time_gran" label="time_granularity">

<VersionBlock firstVersion="1.9">

`time_granularity` は、時間ディメンションの粒度を指定します。
MetricFlow は、基になる列を指定された粒度に変換します。
たとえば、時間ディメンション列に時間単位の粒度を追加すると、MetricFlow は `date_trunc` 関数を実行してタイムスタンプを時間単位に変換します。
クエリ時に時間粒度を簡単に変更し、より粗い粒度に集計できます。たとえば、時間単位から月単位に集計できます。
ただし、粗い粒度からより細かい粒度（月単位から時間単位）に変更することはできません。

サポートされている粒度は次のとおりです:
* nanosecond (Snowflake only)
* microsecond 
* millisecond
* second
* minute
* hour
* day
* week
* month
* quarter
* year

粒度の異なるメトリクス間の集計が可能です。セマンティックレイヤーは、デフォルトで最も粗い粒度で結果を返します。
例えば、日次と月次の粒度で2つのメトリクスをクエリした場合、結果として得られる集計は月次レベルになります。

```yaml
dimensions: 
  - name: created_at
    type: time
    label: "Date of creation"
    expr: ts_created # ts_created is the underlying column name from the table 
    is_partition: True 
    type_params:
      time_granularity: hour 
  - name: deleted_at
    type: time
    label: "Date of deletion"
    expr: ts_deleted # ts_deleted is the underlying column name from the table 
    is_partition: True 
    type_params:
      time_granularity: day 

measures:
  - name: users_deleted
    expr: 1
    agg: sum 
    agg_time_dimension: deleted_at
  - name: users_created
    expr: 1
    agg: sum
```

</VersionBlock>

<VersionBlock lastVersion="1.8">

`time_granularity` specifies the grain of a time dimension. MetricFlow will transform the underlying column to the specified granularity. For example, if you add daily granularity to a time dimension column, MetricFlow will run a `date_trunc` function to convert the timestamp to daily. You can easily change the time grain at query time and aggregate it to a coarser grain, for example, from daily to monthly. However, you can't go from a coarser grain to a finer grain (monthly to daily).

Our supported granularities are:
* day
* week
* month
* quarter
* year

Aggregation between metrics with different granularities is possible, with the Semantic Layer returning results at the coarsest granularity by default. For example, when querying two metrics with daily and monthly granularity, the resulting aggregation will be at the monthly level.

```yaml
dimensions: 
  - name: created_at
    type: time
    label: "Date of creation"
    expr: ts_created # ts_created is the underlying column name from the table 
    is_partition: True 
    type_params:
      time_granularity: day 
  - name: deleted_at
    type: time
    label: "Date of deletion"
    expr: ts_deleted # ts_deleted is the underlying column name from the table 
    is_partition: True 
    type_params:
      time_granularity: day 

measures:
  - name: users_deleted
    expr: 1
    agg: sum 
    agg_time_dimension: deleted_at
  - name: users_created
    expr: 1
    agg: sum
```

</VersionBlock>

</TabItem>

</Tabs>

### SCD Type II

:::caution
現在、SCD タイプ II ディメンションを持つセマンティック モデルにはメジャーを含めることができません。
:::

MetricFlowは、緩やかに変化するディメンション（SCD）タイプIIテーブル上に構築されたセマンティックモデル内のディメンション値に対する結合をサポートしています。
これは、顧客の国別の売上の履歴傾向など、時間の経過とともに変化するグループごとに特定のメトリックをスライスする必要がある場合に便利です。

#### 基本構造

SCD タイプ II は、より粗い時間粒度で値が変化するグループです。
SCD タイプ II テーブルには通常、ディメンションの有効期間を示す 2 つの時間列（「valid_from」（または「tier_start」）と「valid_to」（または「tier_end」））があります。
これにより、メトリックまたはメジャーに対して、異なるディメンション値を持つ有効な行の範囲が作成されます。

MetricFlow は、1 か月などのより粗い時間枠内で利用可能な最も古いディメンション値にメトリックを関連付けます。

デフォルトでは、この時間粒度の開始時に有効なグループが使用されます。

MetricFlow は、SCD タイプ II データ プラットフォーム テーブルの次の基本構造をサポートしています:

| entity_key | dimensions_1 | dimensions_2 | ... | dimensions_x | valid_from | valid_to |
|------------|-------------|-------------|-----|-------------|------------|----------|  

* `entity_key` (必須): テーブル内の各行の一意の識別子（主キーやエンティティ固有の一意の識別子など）。
* `valid_from` (必須): ディメンションの有効期間の開始日を表すタイムスタンプ。セマンティックモデルで `validity_params: is_start: True` を使用して指定します。
* `valid_to` (必須): ディメンションの有効期間の終了日を表すタイムスタンプ。セマンティックモデルで `validity_params: is_end: True` を使用して指定します。

#### セマンティックモデルのパラメータとキー
セマンティックモデルでSCDタイプIIテーブルを構成する際は、`validity_params`を使用して、各ディメンションの有効期間の開始（`valid_from`）と終了（`valid_to`）を指定します。

- `validity_params`: 有効期間を定義するパラメータ。
  - `is_start: True`: 有効期間の開始を示します。SCDテーブルでは`valid_from`として表示されます。
  - `is_end: True`: 有効期間の終了を示します。SCDテーブルでは`valid_to`として表示されます。

設定例を以下に示します。

```yaml
- name: tier_start #  The name of the dimension.
  type: time # The type of dimension (such as time)
  label: "Start date of tier" # A readable label for the dimension
  expr: start_date # Expression or column name the dimension represents
  type_params: # Additional parameters for the dimension type
    time_granularity: day # Specifies the granularity of the time dimension (such as day)
    validity_params: # Defines the validity window
      is_start: True # Indicates the start of the validity period. 
- name: tier_end 
  type: time
  label: "End date of tier"
  expr: end_date
  type_params:
    time_granularity: day
    validity_params:
      is_end: True # Indicates the end of the validity period.
```

SCD タイプ II テーブルには、開始日と終了日を持つ特定のディメンションがあります。
テーブルを結合するには、次の手順に従ってください。
- 追加の [entity `type`](/docs/build/entities#entity-types) パラメータを `natural` キーに設定します。
- `natural` キーを [entity `type`](/docs/build/entities#entity-types) として使用します。つまり、`primary` キーは必要ありません。
- ほとんどの場合、SCD テーブルには論理的に使用可能な `primary` キーがありません。これは、`natural` キーが複数の行にマッピングされるためです。

#### 実装

SCD タイプ II テーブルを実装する際のガイドラインを以下に示します。

- SCD テーブルには、論理構造である `valid_to` および `valid_from` 時間ディメンションが必要です。
- `valid_from` プロパティと `valid_to` プロパティは、SCD テーブル設定ごとに 1 つだけ指定する必要があります。
- `valid_from` プロパティと `valid_to` プロパティは、同じ時間ディメンションで使用または指定しないでください。
- `valid_from` および `valid_to` 時間ディメンションは、重複しない期間をカバーし、各自然キー値に 1 つの行が一致するようにする必要があります（つまり、重複してはならず、異なる値である必要があります）。
- 基盤となる dbt モデルは、[dbt スナップショット](/docs/build/snapshots) を使用して定義することをお勧めします。これにより、SCD タイプ II テーブルレイアウトがサポートされ、テーブルが最新のデータで更新されます。

これは、`entity_key` 列と `timestamp` 列で構成される主キーを使用して、`num_events` というサンプル メトリックがバージョン管理されたディメンション データ (`scd_dimensions` というテーブルに保存されている) と結合される方法を示す SQL コードの例です。

```sql
select metric_time, dimensions_1, sum(1) as num_events
from events a
left outer join scd_dimensions b
on 
  a.entity_key = b.entity_key 
  and a.metric_time >= b.valid_from 
  and (a.metric_time < b. valid_to or b.valid_to is null)
group by 1, 2
```

#### SCD の例

以下は、セマンティックモデルで SCD タイプ II テーブルを使用する例です:

<Expandable alt_header="販売階層とその階層の時間の長さの SCD ディメンション。">

この例では、セマンティックモデルを使用して緩やかに変化するディメンション（SCD）を作成する方法を示します。
SCDテーブルには、営業担当者の階層とその階層の期間に関する情報が含まれています。
次のようなSCDテーブルがあるとします:

| sales_person_id | tier | start_date | end_date | 
|-----------------|------|------------|----------|
| 111             | 1    | 2019-02-03 | 2020-01-05| 
| 111             | 2    | 2020-01-05 | 2048-01-01| 
| 222             | 2    | 2020-03-05 | 2048-01-01| 
| 333             | 2    | 2020-08-19 | 2021-10-22| 
| 333             | 3    | 2021-10-22 | 2048-01-01|  

前述のように、`validity_params` には、SCD テーブル内の各階層またはディメンションの開始日と終了日（またはタイムスタンプ）を示す列を指定する 2 つの重要な引数が含まれています。
- `is_start`
- `is_end`

さらに、エンティティは `natural` としてタグ付けされ、`primary` エンティティと区別されます。`primary` エンティティでは、各エンティティ値に 1 行が割り当てられます。一方、`natural` エンティティでは、エンティティ値とその有効期間の組み合わせごとに 1 行が割り当てられます。

```yaml 
semantic_models:
  - name: sales_person_tiers
    description: SCD Type II table of tiers for salespeople 
    model: {{ ref('sales_person_tiers') }}
    defaults:
      agg_time_dimension: tier_start

    dimensions:
      - name: tier_start
        type: time
        label: "Start date of tier"
        expr: start_date
        type_params:
          time_granularity: day
          validity_params:
            is_start: True
      - name: tier_end 
        type: time
        label: "End date of tier"
        expr: end_date
        type_params:
          time_granularity: day
          validity_params:
            is_end: True
      - name: tier
        type: categorical

    primary_entity: sales_person

    entities:
      - name: sales_person
        type: natural 
        expr: sales_person_id
```

次のコードは、`transactions` のファクト テーブルを保持する別のセマンティック モデルを表しています:

```yaml
semantic_models: 
  - name: transactions 
    description: |
      Each row represents one transaction.
      There is a transaction, product, sales_person, and customer id for 
      every transaction. There is only one transaction id per 
      transaction. The `metric_time` or date is reflected in UTC.
    model: {{ ref('fact_transactions') }}
    defaults:
      agg_time_dimension: metric_time

    entities:
      - name: transaction_id
        type: primary
      - name: customer
        type: foreign
        expr: customer_id
      - name: product
        type: foreign
        expr: product_id
      - name: sales_person
        type: foreign
        expr: sales_person_id

    measures:
      - name: transactions
        expr: 1
        agg: sum
      - name: gross_sales
        expr: sales_price
        agg: sum
      - name: sales_persons_with_a_sale
        expr: sales_person_id
        agg: count_distinct

    dimensions:
      - name: metric_time
        type: time
        label: "Date of transaction"
        is_partition: true
        type_params:
          time_granularity: day
      - name: sales_geo
        type: categorical
```

これで、緩やかに変化する「tier」ディメンションによって整理された「transactions」セマンティックモデルのメトリクスにアクセスできるようになりました。

例えば、営業担当者が2022年3月1日から2022年3月12日までTier 1に所属し、2022年3月12日以降にTier 2に昇格した場合、営業担当者が2022年3月12日にTier 2に昇格したにもかかわらず、Tier 1のディメンション値がそれより前（デフォルトの開始点）であるため、3月以降のすべての取引はTier 1に分類されます。

</Expandable>

<Expandable alt_header="販売階層と、階層が欠落している場合の月別のグループ取引を含む SCD ディメンション">

この例では、セマンティックモデルを用いて緩やかに変化するディメンション（SCD）を作成する方法を示します。
SCDテーブルには、営業担当者の階層とその階層の期間に関する情報が含まれています。
以下のSCDテーブルがあるとします:

| sales_person_id | tier | start_date | end_date | 
|-----------------|------|------------|----------|
| 111             | 1    | 2019-02-03 | 2020-01-05| 
| 111             | 2    | 2020-01-05 | 2048-01-01| 
| 222             | 2    | 2020-03-05 | 2048-01-01| 
| 333             | 2    | 2020-08-19 | 2021-10-22| 
| 333             | 3    | 2021-10-22 | 2048-01-01|  

営業階層の例では、sales_person_id 456 が 2022-03-08 以降は Tier 2 であるものの、2022-03-01 から 2022-03-08 まではこの人物に関連付けられた Tier レベルのディメンションが存在しない場合、Tier 2 より前には Tier が存在しないため、3 月の sales_person_id 456 に関連付けられたすべてのトランザクションは「NA」にグループ化されます。

次のコマンドまたはコードは、各営業階層で生成されたトランザクション数を月ごとに返す方法を示しています:

```bash
# dbt Cloud users
dbt sl query --metrics transactions --group-by metric_time__month,sales_person__tier --order-by metric_time__month,sales_person__tier

# dbt Core users
mf query --metrics transactions --group-by metric_time__month,sales_person__tier --order-by metric_time__month,sales_person__tier

```

</Expandable>
