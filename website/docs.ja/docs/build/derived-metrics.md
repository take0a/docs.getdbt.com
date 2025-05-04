---
title: "派生メトリック"
id: derived
description: "派生メトリックは、他のメトリックの式として定義されます。"
sidebar_label: Derived
tags: [Metrics, Semantic Layer]
---

MetricFlow では、派生メトリクスとは、他のメトリクスを用いた式を定義することで作成されるメトリクスです。これにより、既存のメトリクスを使った計算が可能になります。これは、メトリクスを組み合わせたり、集計された列に対して数学関数を実行したりする場合（例えば、利益メトリクスの作成など）に役立ちます。

派生メトリクスのパラメータ、説明、およびタイプは次のとおりです:

| Parameter | Description | Required | Type | 
| --------- | ----------- | ---- | ---- |
| `name` | メトリックの名前。 | Required | String |  
| `description` | メトリックの説明。 | Optional | String |
| `type` | メトリックのタイプ (累積、派生、比率、または単純)。 | Required | String |  
| `label` | 下流ツールでの表示値を定義します。プレーンテキスト、スペース、引用符（例：`orders_total` または `"orders_total"`）が使用できます。 | Required | String |
| `type_params` | メトリックのタイプパラメータ。 | Required | Dict |  
| `expr` | 派生式。派生メトリックに `expr` がない場合、または `expr` がすべての入力メトリックを使用していない場合、検証警告が表示されます。 | Required | String |
| `metrics` |  派生メトリックで使用されるメトリックのリスト。各エントリには、`alias`、`filter`、`offset_window` などのオプションフィールドを含めることができます。 | Required  | List |  
| `alias` | `expr` で使用できるメトリックのオプションのエイリアス。 | Optional | String |
| `filter` | メトリックに適用するオプションのフィルター。 | Optional | String |  
| `offset_window` | オフセットウィンドウの期間（例：1か月）を設定します。これにより、メトリックの時刻から1か月後のメトリックの値が返されます。  | Optional | String |


以下に、派生メトリックの完全な仕様と例を示します。

```yaml
metrics:
  - name: the metric name # Required
    description: the metric description # Optional
    type: derived # Required
    label: The value that will be displayed in downstream tools #Required
    type_params: # Required
      expr: the derived expression # Required
      metrics: # The list of metrics used in the derived metrics # Required
        - name: the name of the metrics. must reference a metric you have already defined # Required
          alias: optional alias for the metric that you can use in the expr # Optional
          filter: optional filter to apply to the metric # Optional
          offset_window: set the period for the offset window, such as 1 month. This will return the value of the metric one month from the metric time. # Optional
```

高度なデータ モデリングでは、`fill_nulls_with` と `join_to_timespine` を使用して [null メトリック値をゼロに設定](/docs/build/fill-nulls-advanced) し、すべてのデータ行に数値が確保されるようにすることができます。

## 派生メトリックの例

```yaml
metrics:
  - name: order_gross_profit
    description: Gross profit from each order.
    type: derived
    label: Order gross profit
    type_params:
      expr: revenue - cost
      metrics:
        - name: order_total
          alias: revenue
        - name: order_cost
          alias: cost
  - name: food_order_gross_profit
    label: Food order gross profit
    description: "The gross profit for each food order."
    type: derived
    type_params:
      expr: revenue - cost
      metrics:
        - name: order_total
          alias: revenue
          filter: |
            {{ Dimension('order__is_food_order') }} = True
        - name: order_cost
          alias: cost
          filter: |
            {{ Dimension('order__is_food_order') }} = True
  - name: order_total_growth_mom
    description: "Percentage growth of orders total completed to 1 month ago"
    type: derived
    label: Order total growth % M/M
    type_params:
      expr: (order_total - order_total_prev_month)*100/order_total_prev_month
      metrics: 
        - name: order_total
        - name: order_total
          offset_window: 1 month
          alias: order_total_prev_month
```

## 派生メトリックのオフセット

過去の期間の指標の値を使用して計算を実行するには、派生メトリックにオフセットパラメータを追加します。たとえば、前期比成長率を計算したり、ユーザー維持率を追跡したりする場合、この指標オフセットを使用できます。

**注:** オフセットウィンドウを使用して派生メトリックをクエリする場合は、[`metric_time` ディメンション](/docs/build/dimensions#time)を含める必要があります。

次の例は、1か月のオフセットウィンドウを使用して月間収益成長率を計算する方法を示しています。

```yaml
- name: customer_retention
  description: Percentage of customers that are active now and those active 1 month ago
  label: customer_retention
  type_params:
    expr: (active_customers/ active_customers_prev_month)
    metrics:
      - name: active_customers
        alias: current_active_customers
      - name: active_customers
        offset_window: 1 month
        alias: active_customers_prev_month
```

### オフセットウィンドウと粒度

粒度とオフセットウィンドウの組み合わせは任意に指定できます。次の例では、7日間のオフセットと月単位の粒度でメトリックをクエリしています:

```yaml
- name: d7_booking_change
  description: Difference between bookings now and 7 days ago
  type: derived
  label: d7 bookings change
  type_params:
    expr: bookings - bookings_7_days_ago
    metrics:
      - name: bookings
        alias: current_bookings
      - name: bookings
        offset_window: 7 days
        alias: bookings_7_days_ago
```

クエリ「dbt sl query --metrics d7_booking_change --group-by metric_time__month」を実行してメトリクスを計算すると、以下のように計算されます。dbt Core の場合は、「mf query」プレフィックスを使用できます。

1. 指定されたメジャーとディメンションを含む、未集計の生のデータセットを、最小の詳細レベル（現在は「日」）で取得します。
2. 次に、日次データセットに対してオフセット結合を実行し、日付の切り捨てと、要求された粒度への集計を実行します。
例えば、2017年7月の「d7_booking_change」を計算するには、以下の手順に従います。
  - まず、7月の各日の予約額をすべて合計して、予約指標を計算します。
  - 次の表は、この月次集計を構成する日の範囲を示しています。

|   | Orders | Metric_time |
| - | ---- | -------- |
|   | 330 | 2017-07-31 |
|   | 7030 | 2017-07-30 to 2017-07-02 |
|   | 78 | 2017-07-01 |
| Total  | 7438 | 2017-07-01 |

3. 7日間のオフセットを適用して、7月の予約数を計算します。以下の表は、この月次集計を構成する日数の範囲を示しています。月は7日後の2017年7月24日（7日間のオフセット）から始まることに注意してください。

|   | Orders | Metric_time |
| - | ---- | -------- |
|   | 329 | 2017-07-24 |
|   | 6840 | 2017-07-23  to 2017-06-30 |
|   | 83 | 2017-06-24 |
| Total  | 7252 | 2017-07-01 |

4. 最後に、派生メトリックを計算し、最終結果セットを返します:

```bash
bookings - bookings_7_days_ago would be compile as 7438 - 7252 = 186. 
```

| d7_booking_change | metric_time__month |
| ----------------- | ------------------ |
| 186 | 2017-07-01 |

## 関連ドキュメント
- [単純メトリック、派生メトリック、または比率メトリックのnull値を埋める](/docs/build/fill-nulls-advanced)

