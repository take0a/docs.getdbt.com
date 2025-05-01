---
title: Measures
id: measures
description: "メジャーは、モデル内の列に対して実行される集計です。"
sidebar_label: "Measures"
tags: [Metrics, Semantic Layer]
---

メジャーとは、モデル内の列に対して実行される集計です。最終的な指標として使用することも、より複雑な指標の構成要素として使用することもできます。

メジャーには複数の入力があり、それらのフィールドタイプと合わせて次の表で説明します。

import MeasuresParameters from '/snippets/_sl-measures-parameters.md';

<MeasuresParameters />

## メジャー仕様

YAML 形式のメジャー仕様の完全な例を以下に示します。
メジャーの実際の構成は、使用する集計方法によって異なります。

<VersionBlock firstVersion="1.9">

```yaml
semantic_models:
  - name: semantic_model_name
   ..rest of the semantic model config
    measures:
      - name: The name of the measure
        description: 'same as always' ## Optional
        agg: the aggregation type.
        expr: the field
        agg_params: 'specific aggregation properties such as a percentile'  ## Optional
        agg_time_dimension: The time field. Defaults to the default agg time dimension for the semantic model. ##  Optional
        non_additive_dimension: 'Use these configs when you need non-additive dimensions.' ## Optional
        [config](/reference/resource-properties/config): Use the config property to specify configurations for your measure.  ## Optional
          [meta](/reference/resource-configs/meta):  {<dictionary>} Set metadata for a resource and organize resources. Accepts plain text, spaces, and quotes. ## Optional
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
semantic_models:
  - name: semantic_model_name
   ..rest of the semantic model config
    measures:
      - name: The name of the measure
        description: 'same as always' ## Optional
        agg: the aggregation type.
        expr: the field
        agg_params: 'specific aggregation properties such as a percentile'  ## Optional
        agg_time_dimension: The time field. Defaults to the default agg time dimension for the semantic model. ##  Optional
        non_additive_dimension: 'Use these configs when you need non-additive dimensions.' ## Optional
```
</VersionBlock>

### Name

メジャーを作成する際は、カスタム名を付けることも、データプラットフォーム列の「name」を直接使用することもできます。メジャーの「name」が列名と異なる場合は、「expr」を追加して列名を指定する必要があります。メジャーの「name」は、メトリックを作成する際に使用されます。

メジャー名は、プロジェクト内のすべてのセマンティックモデルで一意である必要があり、同じモデル内の既存の「entity」または「dimension」と同じ名前にすることはできません。

### Description

説明は、計算されたメジャーについて説明します。このフィールドには、詳細かつ人間が判読できる説明を入力することを強くお勧めします。

### Aggregation

集計方法は、フィールドの集計方法を決定します。例えば、粒度が「日」の「合計」集計タイプは、特定の日における値を合計します。

サポートされている集計方法は次のとおりです:

| Aggregation types | Description              |
|-------------------|--------------------------|
| sum               | 値の合計 |
| min               | 値全体の最小値 |
| max               | 値全体の最大値 |
| average           | 値の平均 |
| sum_boolean       | ブール型の合計 |
| count_distinct    | 値の個別のカウント |
| median            | 値全体の中央値（p50）の計算 |
| percentile        | 値全体のパーセンタイル計算。 |

#### Percentile aggregation example

`percentile` 集計を使用する場合は、`agg_params` フィールドを使用してパーセンタイル集計の詳細 (計算するパーセンタイルや、離散計算と連続計算のどちらを使用するかなど) を指定する必要があります。

```yaml
name: p99_transaction_value
description: The 99th percentile transaction value
expr: transaction_amount_usd
agg: percentile
agg_params:
  percentile: .99
  use_discrete_percentile: False  # False calculates the continuous percentile, True calculates the discrete percentile.
```

#### Percentile across supported engine types

次の表は、連続パーセンタイル、離散パーセンタイル、近似パーセンタイル、連続パーセンタイル、および近似離散パーセンタイルをサポートする SQL エンジンを示しています。

|  | Cont. | Disc. | Approx. cont | Approx. disc |
| -- | -- | -- | -- | -- |
|Snowflake | [Yes](https://docs.snowflake.com/en/sql-reference/functions/percentile_cont.html) | [Yes](https://docs.snowflake.com/en/sql-reference/functions/percentile_disc.html) | [Yes](https://docs.snowflake.com/en/sql-reference/functions/approx_percentile.html) (t-digest) | No |
| Bigquery | No (window) | No (window) | [Yes](https://cloud.google.com/bigquery/docs/reference/standard-sql/functions-and-operators#approx_quantiles) | No |
| Databricks | [Yes](https://docs.databricks.com/sql/language-manual/functions/percentile_cont.html) | [No](https://docs.databricks.com/sql/language-manual/functions/percentile_disc.html) | No | [Yes](https://docs.databricks.com/sql/language-manual/functions/approx_percentile.html) |
| Redshift | [Yes](https://docs.aws.amazon.com/redshift/latest/dg/r_PERCENTILE_CONT.html) | No (window) | No | [Yes](https://docs.aws.amazon.com/redshift/latest/dg/r_APPROXIMATE_PERCENTILE_DISC.html) |
| [Postgres](https://www.postgresql.org/docs/9.4/functions-aggregate.html) | Yes | Yes | No | No |
| [DuckDB](https://duckdb.org/docs/sql/aggregates.html) | Yes | Yes | Yes (t-digest) | No |

### Expr

メジャーに指定した「name」がモデル内の列名と一致しない場合は、代わりに「expr」パラメータを使用できます。これにより、有効なSQLを使用して、基になる列名を操作し、特定の出力を作成できます。「name」パラメータは、メジャーのエイリアスとして機能します。

**注**: 「expr」パラメータでSQL関数を使用する場合は、**必ずデータプラットフォーム固有のSQLを使用してください**。出力はデータプラットフォームによって異なる可能性があるためです。

:::tip Snowflake ユーザーの皆様へ
Snowflake ユーザーの皆様、`expr` パラメータで週レベルの関数を使用すると、ISO 標準に基づき、デフォルトの週開始日として月曜日が返されるようになりました。アカ​​ウントレベルまたはセッションレベルで `WEEK_START` パラメータを 0 または 1 以外の値に固定するオーバーライドを設定している場合でも、週の開始日は月曜日になります。

`expr` パラメータで `dayofweek` 関数を使用し、Snowflake の従来のデフォルト値である `WEEK_START = 0` を使用すると、Snowflake の従来のデフォルト値である 0 (月曜日) から 6 (日曜日) ではなく、ISO 標準値の 1 (月曜日) から 7 (日曜日) が返されるようになりました。
:::


### Model with different aggregations

<VersionBlock firstVersion="1.9">

```yaml
semantic_models:
  - name: transactions
    description: A record of every transaction that takes place. Carts are considered  multiple transactions for each sku.
    model: ref('schema.transactions')
    defaults:
      agg_time_dimension: transaction_date

# --- entities ---
    entities:
      - name: transaction_id
        type: primary
      - name: customer_id
        type: foreign
      - name: store_id
        type: foreign
      - name: product_id
        type: foreign

# --- measures ---
    measures:
      - name: transaction_amount_usd
        description: Total usd value of transactions
        expr: transaction_amount_usd
        agg: sum
        config:
          meta:
            used_in_reporting: true
      - name: transaction_amount_usd_avg
        description: Average usd value of transactions
        expr: transaction_amount_usd
        agg: average
      - name: transaction_amount_usd_max
        description: Maximum usd value of transactions
        expr: transaction_amount_usd
        agg: max
      - name: transaction_amount_usd_min
        description: Minimum usd value of transactions
        expr: transaction_amount_usd
        agg: min
      - name: quick_buy_transactions 
        description: The total transactions bought as quick buy
        expr: quick_buy_flag 
        agg: sum_boolean 
      - name: distinct_transactions_count
        description: Distinct count of transactions 
        expr: transaction_id
        agg: count_distinct
      - name: transaction_amount_avg 
        description: The average value of transactions 
        expr: transaction_amount_usd
        agg: average 
      - name: transactions_amount_usd_valid # Notice here how we use expr to compute the aggregation based on a condition
        description: The total usd value of valid transactions only
        expr: case when is_valid = True then transaction_amount_usd else 0 end 
        agg: sum
      - name: transactions
        description: The average value of transactions.
        expr: transaction_amount_usd
        agg: average
      - name: p99_transaction_value
        description: The 99th percentile transaction value
        expr: transaction_amount_usd
        agg: percentile
        agg_params:
          percentile: .99
          use_discrete_percentile: False # False calculates the continuous percentile, True calculates the discrete percentile.
      - name: median_transaction_value
        description: The median transaction value
        expr: transaction_amount_usd
        agg: median
        
# --- dimensions ---
    dimensions:
      - name: transaction_date
        type: time
        expr: date_trunc('day', ts) # expr refers to underlying column ts
        type_params:
          time_granularity: day
      - name: is_bulk_transaction
        type: categorical
        expr: case when quantity > 10 then true else false end

```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
semantic_models:
  - name: transactions
    description: A record of every transaction that takes place. Carts are considered  multiple transactions for each sku.
    model: ref('schema.transactions')
    defaults:
      agg_time_dimension: transaction_date

# --- entities ---
    entities:
      - name: transaction_id
        type: primary
      - name: customer_id
        type: foreign
      - name: store_id
        type: foreign
      - name: product_id
        type: foreign

# --- measures ---
    measures:
      - name: transaction_amount_usd
        description: Total usd value of transactions
        expr: transaction_amount_usd
        agg: sum
      - name: transaction_amount_usd_avg
        description: Average usd value of transactions
        expr: transaction_amount_usd
        agg: average
      - name: transaction_amount_usd_max
        description: Maximum usd value of transactions
        expr: transaction_amount_usd
        agg: max
      - name: transaction_amount_usd_min
        description: Minimum usd value of transactions
        expr: transaction_amount_usd
        agg: min
      - name: quick_buy_transactions 
        description: The total transactions bought as quick buy
        expr: quick_buy_flag 
        agg: sum_boolean 
      - name: distinct_transactions_count
        description: Distinct count of transactions 
        expr: transaction_id
        agg: count_distinct
      - name: transaction_amount_avg 
        description: The average value of transactions 
        expr: transaction_amount_usd
        agg: average 
      - name: transactions_amount_usd_valid # Notice here how we use expr to compute the aggregation based on a condition
        description: The total usd value of valid transactions only
        expr: case when is_valid = True then transaction_amount_usd else 0 end 
        agg: sum
      - name: transactions
        description: The average value of transactions.
        expr: transaction_amount_usd
        agg: average
      - name: p99_transaction_value
        description: The 99th percentile transaction value
        expr: transaction_amount_usd
        agg: percentile
        agg_params:
          percentile: .99
          use_discrete_percentile: False # False calculates the continuous percentile, True calculates the discrete percentile.
      - name: median_transaction_value
        description: The median transaction value
        expr: transaction_amount_usd
        agg: median
        
# --- dimensions ---
    dimensions:
      - name: transaction_date
        type: time
        expr: date_trunc('day', ts) # expr refers to underlying column ts
        type_params:
          time_granularity: day
      - name: is_bulk_transaction
        type: categorical
        expr: case when quantity > 10 then true else false end

```
</VersionBlock>

### 非加算ディメンション

一部のメジャーは、時間などの特定のディメンションで集計できません。これは、結果が不正確になる可能性があるためです。例えば、銀行口座の残高のように月次で繰り越すのが適切でないデータや、日次で発生する経常収益を合計しても月次経常収益にならない月次経常収益などが挙げられます。このような状況に対処するために、特定のディメンションを集計から除外する非加算ディメンションを指定できます。

非加算メジャーの設定例を説明するために、登録ユーザーの日付ごとに1行、ユーザーのアクティブなサブスクリプションプラン、およびプランのサブスクリプション額（収益）を含む、以下の列を持つサブスクリプションテーブルを考えてみましょう。

- `date_transaction`: 日次日付。
- `user_id`: 登録ユーザーのID。
- `subscription_plan`: サブスクリプションプランIDを示す列。
- `subscription_value`: 特定のサブスクリプションプランIDの月次サブスクリプション額（収益）を示す列。

`non_additive_dimension` の下のパラメータは、メジャーを集計しないディメンションを指定します。

| Parameter | Description | Field type |
| --- | --- | --- |
| `name`| これは、メジャーを集計しない時間ディメンション (データ ソースで既に定義されている) の名前になります。 | Required |
| `window_choice` | 「min」または「max」のいずれかを選択します。「min」は期間の開始を反映し、「max」は期間の終了を反映します。 | Required |
| `window_groupings` | グループ化するエンティティを指定します。 | Optional |


```yaml
semantic_models:
  - name: subscriptions
    description: A subscription table with one row per date for each active user and their subscription plans. 
    model: ref('your_schema.subscription_table')
    defaults:
      agg_time_dimension: subscription_date

    entities:
      - name: user_id
        type: foreign
    primary_entity: subscription

    dimensions:
      - name: subscription_date
        type: time
        expr: date_transaction
        type_params:
          time_granularity: day

    measures: 
      - name: count_users
        description: Count of users at the end of the month 
        expr: user_id
        agg: count_distinct
        non_additive_dimension: 
          name: subscription_date
          window_choice: max 
      - name: mrr
        description: Aggregate by summing all users' active subscription plans
        expr: subscription_value
        agg: sum 
        non_additive_dimension: 
          name: subscription_date
          window_choice: max
      - name: user_mrr
        description: Group by user_id to achieve each user's MRR
        expr: subscription_value
        agg: sum  
        non_additive_dimension: 
          name: subscription_date
          window_choice: max
          window_groupings: 
            - user_id 

metrics:
  - name: mrr_metrics
    type: simple
    type_params:
        measure: mrr
```

次の構文を使用して、半加法メトリックをクエリできます:

For dbt Cloud:

```bash
dbt sl query --metrics mrr_by_end_of_month --group-by subscription__subscription_date__month --order subscription__subscription_date__month 
dbt sl query --metrics mrr_by_end_of_month --group-by subscription__subscription_date__week --order subscription__subscription_date__week 
```

For dbt Core:

```bash
mf query --metrics mrr_by_end_of_month --group-by subscription__subscription_date__month --order subscription__subscription_date__month 
mf query --metrics mrr_by_end_of_month --group-by subscription__subscription_date__week --order subscription__subscription_date__week 
```

import SetUpPages from '/snippets/_metrics-dependencies.md';

<SetUpPages /> 
