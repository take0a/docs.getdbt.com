---
id: ratio
title: "比率メトリック"
description: "比率メトリックを使用して、2 つの測定値から比率を作成します。"
sidebar_label: Ratio
tags: [Metrics, Semantic Layer]
---

比率を使用すると、2つの指標の比率を作成できます。
分子と分母の指標を指定するだけです。
さらに、指標を計算する際に制約文字列を使用して、分子と分母の両方にディメンションフィルターを適用することもできます。

比率メトリックのパラメータ、説明、およびタイプは次のとおりです。

| Parameter | Description | Required | Type | 
| --------- | ----------- | ---- | ---- |
| `name` | メトリックの名前。 | Required | String |
| `description` | メトリックの説明。 | Optional | String |
| `type` | メトリックのタイプ (累積、派生、比率、または単純)。 | Required | String |
| `label` | 下流ツールでの表示値を定義します。プレーンテキスト、スペース、引用符（例：`orders_total` または `"orders_total"`）が使用できます。 | Required | String |
| `type_params` | メトリックのタイプパラメータ。 | Required | Dict |
| `numerator` | 分子またはプロパティの構造に使用されるメトリックの名前。 | Required | String or dict |
| `denominator` | 分母に使用されるメトリックの名前、またはプロパティの構造。 | Required  | String  or dict |
| `filter` | 分子または分母のオプション フィルター。 | Optional | String |
| `alias` | 分子または分母のオプションのエイリアス。 | Optional | String |

以下に、比率メトリックの完全な仕様と例を示します。

<File name="models/metrics/file_name.yml">
 
```yaml
metrics:
  - name: The metric name # Required
    description: the metric description # Optional
    type: ratio # Required
    label: String that defines the display value in downstream tools. (such as orders_total or "orders_total") #Required
    type_params: # Required
      numerator: The name of the metric used for the numerator, or structure of properties # Required
        name: Name of metric used for the numerator # Required
        filter: Filter for the numerator # Optional
        alias: Alias for the numerator # Optional
      denominator: The name of the metric used for the denominator, or structure of properties # Required
        name: Name of metric used for the denominator # Required
        filter: Filter for the denominator # Optional
        alias: Alias for the denominator # Optional
```
</File>

高度なデータ モデリングでは、`fill_nulls_with` と `join_to_timespine` を使用して [null メトリック値をゼロに設定](/docs/build/fill-nulls-advanced) し、すべてのデータ行に数値が確保されるようにすることができます。

## 比率メトリックの例

これらの例は、モデル内で比率メトリックを作成する方法を示しています。分子指標と分母指標へのフィルターの適用など、基本的なユースケースと高度なユースケースを網羅しています。

#### 例1
この例は、食品の注文数と総注文数の比率を計算する基本的な比率メトリックです。

<File name="models/metrics/file_name.yml">
 
```yaml
metrics:
  - name: food_order_pct
    description: "The food order count as a ratio of the total order count"
    label: Food order ratio
    type: ratio
    type_params: 
      numerator: food_orders
      denominator: orders
```
</File>

#### 例2
この例は、分子にフィルターとエイリアスを適用し、総注文数に対する料理注文数の比率を計算する比率メトリックです。これらの属性を追加するには、name属性にも明示的なキーを使用する必要があることに注意してください。

<File name="models/metrics/file_name.yml">
 
```yaml
metrics:
  - name: food_order_pct
    description: "The food order count as a ratio of the total order count, filtered by location"
    label: Food order ratio by location
    type: ratio
    type_params:
      numerator:
        name: food_orders
        filter: location = 'New York'
        alias: ny_food_orders
      denominator:
        name: orders
        filter: location = 'New York'
        alias: ny_orders
```
</File>

## 異なるセマンティックモデルを用いた比率メトリック

システムは、サブクエリで分子と分母の値を計算することで、異なるセマンティックモデルから得られた比率メトリックを簡略化し、1つの比率メトリックに変換します。その後、共通のディメンションに基づいて結果セットを結合し、最終的な比率を計算します。このような比率メトリック用に生成されたSQLの例を以下に示します。


```sql
select
  subq_15577.metric_time as metric_time,
  cast(subq_15577.mql_queries_created_test as double) / cast(nullif(subq_15582.distinct_query_users, 0) as double) as mql_queries_per_active_user
from (
  select
    metric_time,
    sum(mql_queries_created_test) as mql_queries_created_test
  from (
    select
      cast(query_created_at as date) as metric_time,
      case when query_status in ('PENDING','MODE') then 1 else 0 end as mql_queries_created_test
    from prod_dbt.mql_query_base mql_queries_test_src_2552 
  ) subq_15576
  group by
    metric_time
) subq_15577
inner join (
  select
    metric_time,
    count(distinct distinct_query_users) as distinct_query_users
  from (
    select
      cast(query_created_at as date) as metric_time,
      case when query_status in ('MODE','PENDING') then email else null end as distinct_query_users
    from prod_dbt.mql_query_base mql_queries_src_2585 
  ) subq_15581
  group by
    metric_time
) subq_15582
on
  (
    (
      subq_15577.metric_time = subq_15582.metric_time
    ) or (
      (
        subq_15577.metric_time is null
      ) and (
        subq_15582.metric_time is null
      )
    )
  )
```

## フィルターを追加

ユーザーは、入力メトリックに直接フィルターを適用することで、比率メトリックの入力メトリックに制約を定義できます。例:

<File name="models/metrics/file_name.yml">
 
```yaml
metrics:
  - name: frequent_purchaser_ratio
    description: Fraction of active users who qualify as frequent purchasers
    type: ratio
    type_params:
      numerator:
        name: distinct_purchasers
        filter: |
          {{Dimension('customer__is_frequent_purchaser')}}
        alias: frequent_purchasers
      denominator:
        name: distinct_purchasers
```
</File>

分子で参照されているメトリックの「filter」パラメータと「alias」パラメータに注意してください。
- 「filter」パラメータを使用して、関連付けられているメトリックにフィルターを適用します。
- 「alias」パラメータは、同じメトリックが異なるフィルターで使用されている場合に、レンダリングされたSQLクエリで名前の競合を回避するために使用されます。
- 名前の競合がない場合、「alias」パラメータは省略できます。

## 関連ドキュメント
- [単純メトリック、派生メトリック、または比率メトリックのnull値を埋める](/docs/build/fill-nulls-advanced)
