---
title: "累積指標"
id: cumulative
description: "累積メトリックを使用して、特定の期間にわたって測定値を集計します。"
sidebar_label: Cumulative
tags: [Metrics, Semantic Layer]
---

累積指標は、指定された累積期間にわたって測定値を集計します。
期間が指定されていない場合、その期間は無限とみなされ、全期間にわたって値が累積されます。
累積指標を追加する前に、[時間スパインモデル](/docs/build/metricflow-time-spine)を作成する必要があります。

累積指標は、週次アクティブユーザー数や月次累計収益などの計算に役立ちます。
累積指標のパラメータ、説明、およびタイプは次のとおりです。

:::tip
パラメータが別のパラメータ内にネストされているかどうかを示すために、二重コロン (::) を使用することに注意してください。
たとえば、`measure::name` は、`name` パラメータが `measure` の下にネストされていることを意味します。
:::

## Parameters

<VersionBlock firstVersion="1.9">

| Parameter   | <div style={{width:'350px'}}>Description</div>   | Required | Type      |
|-------------|---------------------------------------------------|----------|-----------|
| `name`  | メトリックの名前。       | Required  | String |
| `description`       | メトリックの説明。     | Optional  | String |
| `type`    | メトリックのタイプ (累積、派生、比率、または単純)。 | Required  | String |  
| `label`     | 下流ツールでの表示値を定義する必須文字列。プレーンテキスト、スペース、引用符（例：`orders_total`、`"orders_total"`）が使用できます。  | Required  | String |
| `type_params`    | メトリックの型パラメータ。`type_params::measure` のように、二重コロンで示されるネストされたパラメータをサポートします。 | Required  | Dict |
| `type_params::measure`   | メトリックに関連付けられたメジャー。ショートハンド（文字列）とオブジェクト構文の両方をサポートします。ショートハンドは名前のみが必要な場合に使用され、オブジェクト構文では追加の属性を指定できます。 | Required  | Dict |
| `measure::name`    | 参照されるメジャーの名前。`type_params::measure` のオブジェクト構文を使用する場合は必須です。 | Optional  | String |
| `measure::fill_nulls_with`     | メトリック定義内の null を置き換える値 (たとえば、0) を設定します。 | Optional  | Integer or string |
| `measure::join_to_timespine` | 集計されたメジャーをタイムスパインテーブルに結合して、欠落している日付を埋めるかどうかを示すブール値。デフォルトは「false」です。 | Optional  | Boolean |
| `type_params::cumulative_type_params`     | 累積メトリックの `window`、`period_agg`、`grain_to_date` などの属性を構成します。 | Optional  | Dict |
| `cumulative_type_params::window`      | 蓄積期間（「1か月」、「7日間」、「1年」など）を指定します。「grain_to_date」と併用することはできません。| Optional  | String |
| `cumulative_type_params::grain_to_date`   | `month` などの累積グレインを設定し、指定されたグレイン期間の開始時に累積を再開します。`window` と併用することはできません。 | Optional  | String |
| `cumulative_type_params::period_agg`  | データを異なる粒度（`first`、`last`、または `average`）で集計する際に、累積メトリックを集計する方法を定義します。`window` が指定されていない場合は、デフォルトで `first` になります。 | Optional  | String |

</VersionBlock>

<VersionBlock lastVersion="1.8">

| Parameter | <div style={{width:'350px'}}>Description</div> | Type |
| --------- | ----------- | ---- |
| `name` | The name of the metric. | Required |
| `description` | The description of the metric. | Optional |
| `type` | The type of the metric (cumulative, derived, ratio, or simple). | Required |
| `label` | Required string that defines the display value in downstream tools. Accepts plain text, spaces, and quotes (such as `orders_total` or `"orders_total"`). | Required |
| `type_params` | The type parameters of the metric. Supports nested parameters indicated by the double colon, such as `type_params::measure`. | Required |
| `window` | The accumulation window, such as `1 month`, `7 days`, or `1 year`. This can't be used with `grain_to_date`. | Optional  |
| `grain_to_date` | Sets the accumulation grain, such as `month`, which will accumulate data for one month and then restart at the beginning of the next. This can't be used with `window`. | Optional |
| `type_params::measure` | A list of measure inputs | Required |
| `measure:name` | The name of the measure being referenced. Required if using object syntax for `type_params::measure`.  | Optional  |
| `measure:fill_nulls_with` | Set the value in your metric definition instead of null (such as zero).| Optional |
| `measure:join_to_timespine` | Boolean that indicates if the aggregated measure should be joined to the time spine table to fill in missing dates. Default `false`. | Optional |

</VersionBlock>

<Expandable alt_header="type_params::measure の説明">
  
`type_params::measure` 設定は、以下の複数の方法で記述できます:
- 省略構文 - メジャー名のみを指定するには、単純な文字列値を使用します。
これは、他の属性が必要ない場合の省略構文です。
  ```yaml
  type_params:
    measure: revenue
  ```
- オブジェクト構文 - メジャーに詳細や属性（フィルターの追加、null 値の処理、タイムスパインへの結合の指定など）を追加するには、オブジェクト構文を使用する必要があります。
これにより、メジャー名だけでなく、追加の構成が可能になります。

  ```yaml
  type_params:
    measure:
      name: order_total
      fill_nulls_with: 0
      join_to_timespine: true
  ```
</Expandable>

### 完全な仕様
以下に、累積指標の完全な仕様と例を示します。

<File name='models/marts/sem_semantic_model_name.yml'>

<VersionBlock firstVersion="1.9">

```yaml
metrics:
  - name: The metric name # Required
    description: The metric description # Optional
    type: cumulative # Required
    label: The value that will be displayed in downstream tools # Required
    type_params: # Required
      cumulative_type_params:
        period_agg: first # Optional. Defaults to first. Accepted values: first|last|average
        window: The accumulation window, such as 1 month, 7 days, 1 year. # Optional. It cannot be used with grain_to_date.
        grain_to_date: Sets the accumulation grain, such as month will accumulate data for one month, then restart at the beginning of the next.  # Optional. It cannot be used with window.
      measure: 
        name: The measure you are referencing. # Required
        fill_nulls_with: Set the value in your metric definition instead of null (such as zero). # Optional
        join_to_timespine: true/false # Boolean that indicates if the aggregated measure should be joined to the time spine table to fill in missing dates. Default `false`. # Optional

```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
metrics:
  - name: The metric name  # Required
    description: The metric description  # Optional
    type: cumulative  # Required
    label: The value that will be displayed in downstream tools  # Required
    type_params:  # Required
      measure: 
        name: The measure you are referencing  # Required
        fill_nulls_with: Set the value in your metric definition instead of null (such as zero)  # Optional
        join_to_timespine: false  # Boolean that indicates if the aggregated measure should be joined to the time spine table to fill in missing dates. Default `false`. # Optional
      window: 1 month  # The accumulation window, such as 1 month, 7 days, 1 year. Optional. Cannot be used with grain_to_date.
      grain_to_date: month  # Sets the accumulation grain, such as month will accumulate data for one month, then restart at the beginning of the next. Optional. Cannot be used with window.
```
</VersionBlock>

</File>

## 累積メトリクスの例

累積メトリクスは、指定された期間のデータを測定します。ウィンドウパラメータが渡されない場合は、期間を無限とみなし、全期間にわたってデータを累積します。

次の例は、YAMLファイルで累積メトリクスを定義する方法を示しています。

<VersionBlock firstVersion="1.9">

- `cumulative_order_total`: 全期間の累計注文合計を計算します。`type params` を使用して、集計対象となるメジャー `order_total` を指定します。

- `cumulative_order_total_l1m`: 過去1か月間の累計注文合計を計算します。`cumulative_type_params` を使用して、`window` に1か月を指定します。

- `cumulative_order_total_mtd`: 月初からの累計注文合計を計算します。`cumulative_type_params` を使用して、`grain_to_date` に`month` を指定します。

</VersionBlock>

<VersionBlock lastVersion="1.8">

- `cumulative_order_total`: Calculates the cumulative order total over all time. Uses `type params` to specify the measure `order_total` to be aggregated.

- `cumulative_order_total_l1m`: Calculates the trailing 1-month cumulative order total. Uses `type params` to specify a `window` of 1 month.

- `cumulative_order_total_mtd`: Calculates the month-to-date cumulative order total, respectively. Uses `type params` to specify a `grain_to_date` of `month`.

</VersionBlock>

<File name='models/marts/sem_semantic_model_name.yml'>

<VersionBlock firstVersion="1.9">

```yaml
metrics:
  - name: cumulative_order_total
    label: Cumulative order total (All-Time)    
    description: The cumulative value of all orders
    type: cumulative
    type_params:
      measure: 
        name: order_total
  
  - name: cumulative_order_total_l1m
    label: Cumulative order total (L1M)   
    description: Trailing 1-month cumulative order total
    type: cumulative
    type_params:
      measure: 
        name: order_total
      cumulative_type_params:
        window: 1 month
  
  - name: cumulative_order_total_mtd
    label: Cumulative order total (MTD)
    description: The month-to-date value of all orders
    type: cumulative
    type_params:
      measure: 
        name: order_total
      cumulative_type_params:
        grain_to_date: month
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
metrics:
  - name: cumulative_order_total
    label: Cumulative order total (All-Time)    
    description: The cumulative value of all orders
    type: cumulative
    type_params:
      measure: 
        name: order_total
  
  - name: cumulative_order_total_l1m
    label: Cumulative order total (L1M)   
    description: Trailing 1-month cumulative order total
    type: cumulative
    type_params:
      measure: 
        name: order_total
      window: 1 month
  
  - name: cumulative_order_total_mtd
    label: Cumulative order total (MTD)
    description: The month-to-date value of all orders
    type: cumulative
    type_params:
      measure: 
        name: order_total
      grain_to_date: month
```
</VersionBlock>

</File>

<VersionBlock firstVersion="1.9">

### 粒度オプション

`period_agg` パラメータを `first()`、`last()`、`average()` 関数と共に使用することで、指定した期間の累積指標を集計できます。これは、累積指標の粒度オプションが他の指標タイプのオプションと異なるためです。
- 他の指標については、粒度を実装するために `date_trunc` 関数を使用します。
- ただし、累積指標は非加算的（値を加算できない）であるため、`date_trunc` 関数を使用して時間粒度の粒度を変更することはできません。
- デフォルトでは、期間の最初の値が取得されます。`period_agg` パラメータを使用して別の関数を指定することで、これを変更できます。

次の例では、すべての注文の累積収益を計算する累積指標 `cumulative_revenue` を定義します。

<File name='models/marts/sem_semantic_model_name.yml'>

```yaml
- name: cumulative_revenue
  description: The cumulative revenue for all orders.
  label: Cumulative revenue (all-time)
  type: cumulative
  type_params:
    measure: revenue
    cumulative_type_params:
      period_agg: first # Optional. Defaults to first. Accepted values: first|end|average
```
</File>

この例では、`period_agg` は `first` に設定されており、選択した粒度ウィンドウの最初の値が選択されます。
`cumulative_revenue` を週単位でクエリするには、次のクエリ構文を使用します。
- `dbt sl query --metrics cumulative_revenue --group-by metric_time__week`

<Expandable alt_header="トグルを展開してSQLのコンパイル方法を表示します">

`first` 値を選択するために `window` 関数を使用していることに注意してください。`last` と `average` については、生成されたSQL内の `first_value()` 関数をそれぞれ `last_value()` と `average` に置き換えます。

```sql
-- re-aggregate metric via the group by
select
  metric_time__week,
  metric_time__quarter,
  revenue_all_time
from (
  -- window function for metric re-aggregation
  select
    metric_time__week,
    metric_time__quarter,
    first_value(revenue_all_time) over (
      partition by
        metric_time__week,
        metric_time__quarter
      order by metric_time__day
      rows between unbounded preceding and unbounded following
    ) as revenue_all_time
  from (
    -- join self over time range
    -- pass only elements: ['txn_revenue', 'metric_time__week', 'metric_time__quarter', 'metric_time__day']
    -- aggregate measures
    -- compute metrics via expressions
    select
      subq_11.metric_time__day as metric_time__day,
      subq_11.metric_time__week as metric_time__week,
      subq_11.metric_time__quarter as metric_time__quarter,
      sum(revenue_src_28000.revenue) as revenue_all_time
    from (
      -- time spine
      select
        ds as metric_time__day,
        date_trunc('week', ds) as metric_time__week,
        date_trunc('quarter', ds) as metric_time__quarter
      from mf_time_spine subq_12
      group by
        ds,
        date_trunc('week', ds),
        date_trunc('quarter', ds)
    ) subq_11
    inner join fct_revenue revenue_src_28000
    on (
      date_trunc('day', revenue_src_28000.created_at) <= subq_11.metric_time__day
    )
    group by
      subq_11.metric_time__day,
      subq_11.metric_time__week,
      subq_11.metric_time__quarter
  ) subq_16
) subq_17
group by
  metric_time__week,
  metric_time__quarter,
  revenue_all_time

```

</Expandable>

</VersionBlock>

### ウィンドウオプション

このセクションでは、ウィンドウオプションを指定する場合と指定しない場合の例について詳しく説明します。

- ウィンドウが指定されている場合、MetricFlow は基になる指標にスライディングウィンドウを適用します。例えば、週単位のアクティブユーザーを 7 日間のウィンドウで追跡します。
- ウィンドウを指定しない場合、累積指標は全期間にわたって値を累積します。これは、現在の収益やアクティブなサブスクリプションなどの累計を計算するのに役立ちます。

<Expandable alt_header="ウィンドウが指定された例">

ウィンドウオプションが指定されている場合、MetricFlow は基になるメジャーにスライディングウィンドウを適用します。

基になるメジャー「customers」が、Jaffle ショップで注文を行う一意の顧客数をカウントするように設定されているとします。

<File name='models/marts/sem_semantic_model_name.yml'>

```yaml
measures:
  - name: customers
    expr: customer_id
    agg: count_distinct

```
</File>

累積メトリック「weekly_customers」は次のように記述できます:

<VersionBlock firstVersion="1.9">

<File name='models/marts/sem_semantic_model_name.yml'>

``` yaml
metrics: 
  - name: weekly_customers # Define the measure and the window.
  type: cumulative
  type_params:
    measure: customers
    cumulative_type_params:
      window: 7 days # Setting the window to 7 days since we want to track weekly active
      period_agg: first # This will choose the first value of the granularity window when changing the granularity.
```
</File>

サンプルYAMLの例から、以下の点に注意してください。

* `type`: 指標の種類を示すために、cumulative を指定します。
* `type_params`: `measure` を指定して累積指標を設定します。
* `cumulative_type_params`: 必要に応じて、`window`、`period_agg`、`grain_to_date` の設定を追加します。

例えば、`weekly_customers` 累積指標では、MetricFlow は関連する顧客の7日間のスライディングウィンドウを取得し、count distinctive 関数を適用します。

`window` を削除すると、指標は全期間にわたって累積されます。
</VersionBlock>

<VersionBlock lastVersion="1.8">

<File name='models/marts/sem_semantic_model_name.yml'>

``` yaml
metrics: 
  - name: weekly_customers # Define the measure and the window.
  type: cumulative
  type_params:
    measure: customers
    window: 7 days # Setting the window to 7 days since we want to track weekly active
```
</File>
</VersionBlock>

サンプル YAML の例から、以下の点に注意してください。

* `type`: 指標の種類を示す累積値を指定します。
* `type_params`: `measure` を指定して累積指標を設定し、オプションで `window` または `grain_to_date` 構成を追加します。

例えば、累積指標 `weekly_customers` では、MetricFlow は関連する顧客の 7 日間のスライディングウィンドウを取得し、count distinctive 関数を適用します。

`window` を削除すると、指標は全期間にわたって累積されます。

</Expandable>

<Expandable alt_header="ウィンドウが指定されていない例">

あなた（この例ではサブスクリプションベースの企業）が、以下の列を持つイベントベースのログテーブルを持っているとします。

* `date`: 日付列
* `user_id`: (整数) イベントの責任を負う各ユーザーに指定されたID
* `subscription_plan`: (整数) ユーザーに関連付けられた特定のサブスクリプションプランを示す列
* `subscription_revenue`: (整数) サブスクリプションプランに関連付けられた値を示す列
* `event_type`: (整数) 追加されたサブスクリプションを示す +1 または削除されたサブスクリプションを示す -1 が設定される列
* `revenue`: (整数) `event_type` と `subscription_revenue` を乗算し、特定の日付における収益の増加または減少を表す列。

ウィンドウを指定せずに累積メトリクスを使用すると、アクティブなサブスクリプションの数や収益などのメトリクスの累積合計を任意の時点で計算できます。
次の YAML ファイルは、現在の収益とアクティブなサブスクリプションの合計数を累積合計として取得するための累積メトリクスの作成方法を示しています:

<File name='models/marts/sem_semantic_model_name.yml'>

```yaml
measures:
  - name: revenue
    description: Total revenue
    agg: sum
    expr: revenue
  - name: subscription_count
    description: Count of active subscriptions
    agg: sum
    expr: event_type
metrics:
  - name: current_revenue
    description: Current revenue
    label: Current Revenue
    type: cumulative
    type_params:
      measure: revenue
  - name: active_subscriptions
    description: Count of active subscriptions
    label: Active Subscriptions
    type: cumulative
    type_params:
      measure: subscription_count

```

</File>
</Expandable>

### 日付までのグレイン

累積メトリック設定で日付までのグレインを指定すると、週、月、年などのグレインの開始時点からメトリックを累積できます。
月などのウィンドウを使用する場合、MetricFlowは1か月分遡ります。
ただし、日付までのグレインでは、データの最新日付に関係なく、常にグレインの先頭から累積が開始されます。

例えば、基礎となるメジャーが「order_total」であるとします:

<File name='models/marts/sem_semantic_model_name.yml'>

```yaml
    measures:
      - name: order_total
        agg: sum
```
</File>

1か月間のウィンドウと月単位のグレイン（現在までのデータ）の違いを比較できます。
- ウィンドウアプローチの累積メトリックは、1か月間のスライディングウィンドウを適用します。
- 月単位のグレイン（現在までのデータ）は、毎月​​初めにリセットされます。

<File name='models/marts/sem_semantic_model_name.yml'>

<VersionBlock firstVersion="1.9">

```yaml
metrics:
  - name: cumulative_order_total_l1m  # For this metric, we use a window of 1 month 
    label: Cumulative order total (L1M)
    description: Trailing 1-month cumulative order amount
    type: cumulative
    type_params:
      measure: order_total
      cumulative_type_params:
        window: 1 month # Applies a sliding window of 1 month
  - name: cumulative_order_total_mtd   # For this metric, we use a monthly grain-to-date 
    label: Cumulative order total (MTD)
    description: The month-to-date value of all orders
    type: cumulative
    type_params:
      measure: order_total
      cumulative_type_params:
        grain_to_date: month # Resets at the beginning of each month
        period_agg: first # Optional. Defaults to first. Accepted values: first|last|average
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
metrics:
  - name: cumulative_order_total_l1m  # For this metric, we use a window of 1 month 
    label: Cumulative order total (L1M)
    description: Trailing 1-month cumulative order amount
    type: cumulative
    type_params:
      measure: order_total
    window: 1 month # Applies a sliding window of 1 month
  - name: cumulative_order_total_mtd   # For this metric, we use a monthly grain-to-date 
    label: Cumulative order total (MTD)
    description: The month-to-date value of all orders
    type: cumulative
    type_params:
      measure: order_total
      grain_to_date: month # Resets at the beginning of each month
```
</VersionBlock>
</File>

現在までのグレインの累積メトリック:

<VersionBlock firstVersion="1.9">
<File name='models/marts/sem_semantic_model_name.yml'>

```yaml
- name: orders_last_month_to_date
  label: Orders month to date
  type: cumulative
  type_params:
    measure: order_count
    cumulative_type_params:
      grain_to_date: month
```
</File>

<Expandable alt_header="Expand toggle to view how the SQL compiles">

```sql
with staging as (
    select
        subq_3.date_day as metric_time__day,
        date_trunc('week', subq_3.date_day) as metric_time__week,
        sum(subq_1.order_count) as orders_last_month_to_date
    from dbt_jstein.metricflow_time_spine subq_3
    inner join (
        select
            date_trunc('day', ordered_at) as metric_time__day,
            1 as order_count
        from analytics.dbt_jstein.orders orders_src_10000
    ) subq_1
    on (
        subq_1.metric_time__day <= subq_3.date_day
    ) and (
        subq_1.metric_time__day >= date_trunc('month', subq_3.date_day)
    )
    group by
        subq_3.date_day,
        date_trunc('week', subq_3.date_day)
)

select
    *
from (
    select
        metric_time__week,
        first_value(orders_last_month_to_date) over (partition by date_trunc('week', metric_time__day) order by metric_time__day) as cumulative_revenue
    from
        staging
)
group by
    metric_time__week,
    cumulative_revenue
order by
    metric_time__week
    1
```

</Expandable>
</VersionBlock>

<VersionBlock lastVersion="1.8">
<File name='models/marts/sem_semantic_model_name.yml'>

```yaml
- name: orders_last_month_to_date
  label: Orders month to date
  type: cumulative
  type_params:
    measure: order_count
    grain_to_date: month
```
</File>
</VersionBlock>

## SQL 実装例

特定の期間におけるメトリックの累積値を計算するには、プライマリ時間ディメンションを結合キーとして、タイムスパインテーブルへの時間範囲結合を実行します。
結合の累積期間を使用して、特定の日にレコードを含めるかどうかを決定します。
以下の SQL コードは、累積メトリックの例から生成されたものです。参考までに示します。

累積メトリックを実装するには、以下の SQL コード例を参照してください。

``` sql
select
  count(distinct distinct_users) as weekly_active_users,
  metric_time
from (
  select
    subq_3.distinct_users as distinct_users,
    subq_3.metric_time as metric_time
  from (
    select
      subq_2.distinct_users as distinct_users,
      subq_1.metric_time as metric_time
    from (
      select
        metric_time
      from transform_prod_schema.mf_time_spine subq_1356
      where (
        metric_time >= cast('2000-01-01' as timestamp)
      ) and (
        metric_time <= cast('2040-12-31' as timestamp)
      )
    ) subq_1
    inner join (
      select
        distinct_users as distinct_users,
        date_trunc('day', ds) as metric_time
      from demo_schema.transactions transactions_src_426
      where (
        (date_trunc('day', ds)) >= cast('1999-12-26' as timestamp)
      ) AND (
        (date_trunc('day', ds)) <= cast('2040-12-31' as timestamp)
      )
    ) subq_2
    on
      (
        subq_2.metric_time <= subq_1.metric_time
      ) and (
        subq_2.metric_time > dateadd(day, -7, subq_1.metric_time)
      )
  ) subq_3
)
group by
  metric_time,
limit 100;

```

## 制限事項

累積指標の定義で「ウィンドウ」を指定する場合、SQLクエリのディメンションとして「メトリック時間」を含める必要があります。これは、累積ウィンドウが指標の時間に基づいているためです。例：

```sql
select
  count(distinct subq_3.distinct_users) as weekly_active_users,
  subq_3.metric_time
from (
  select
    subq_2.distinct_users as distinct_users,
    subq_1.metric_time as metric_time
group by
  subq_3.metric_time
```

## 関連ドキュメント
- [単純指標、派生指標、または比率指標のnull値を埋める](/docs/build/fill-nulls-advanced)
