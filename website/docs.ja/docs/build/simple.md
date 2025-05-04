---
title: "シンプルメトリクス"
id: simple
description: "シンプルメトリクスを使用して、単一の測定値を直接参照します。"
sidebar_label: Simple
tags: [Metrics, Semantic Layer]
pagination_next: null
---

シンプルメトリクスとは、追加のメジャーを介さずに単一のメジャーを直接参照するメトリクスです。データプラットフォーム内の列の集計であり、1つまたは複数のディメンションでフィルタリングできます。

シンプルメトリクスのパラメータ、説明、およびタイプは次のとおりです:

:::tip
パラメータが別のパラメータ内にネストされているかどうかを示すために、二重コロン (::) を使用することに注意してください。たとえば、`query_params::metrics` は、`metrics` パラメータが `query_params` の下にネストされていることを意味します。
:::


| Parameter | Description | Required | Type |
| --------- | ----------- | ---- | ---- |
| `name` | メトリックの名前。 | Required | String |
| `description` | メトリックの説明。 | Optional | String |
| `type` | メトリックのタイプ (累積、派生、比率、または単純)。 | Required | String |
| `label` | 下流ツールでの表示値を定義します。プレーンテキスト、スペース、引用符（例：`orders_total` または `"orders_total"`）が使用できます。 | Required | String |
| `type_params` | メトリックのタイプパラメータ。 | Required | Dict |
| `measure` | 測定入力のリスト。 | Required | List |
| `measure:name` | 参照している測定値。 | Required | String |
| `measure:alias` | メジャーの名前を変更するためのオプションの [`alias`](/reference/resource-configs/alias)。 | Optional | String |
| `measure:filter` | メジャーに適用されるオプションの「フィルター」。 | Optional | String |
| `measure:fill_nulls_with` | メトリック定義で null (ゼロなど) ではなく値を設定します。 | Optional | String |
| `measure:join_to_timespine` | 欠落している日付を埋めるために、集計されたメジャーをタイムスパインテーブルに結合するかどうかを示します。デフォルトは「false」です。 | Optional | Boolean |

以下に、単純なメトリックの完全な仕様と例を示します。

```yaml
metrics:
  - name: The metric name # Required
    description: the metric description # Optional
    type: simple # Required
    label: The value that will be displayed in downstream tools # Required
    type_params: # Required
      measure: 
        name: The name of your measure # Required
        alias: The alias applied to the measure. # Optional
        filter: The filter applied to the measure. # Optional
        fill_nulls_with: Set value instead of null  (such as zero) # Optional
        join_to_timespine: true/false # Boolean that indicates if the aggregated measure should be joined to the time spine table to fill in missing dates. # Optional

```

高度なデータ モデリングでは、`fill_nulls_with` と `join_to_timespine` を使用して [null メトリック値をゼロに設定](/docs/build/fill-nulls-advanced) し、すべてのデータ行に数値が確保されるようにすることができます。

<!-- create_metric not supported yet
:::tip

If you've already defined the measure using the `create_metric: true` parameter, you don't need to create simple metrics. However, if you want to include a filter in the final metric, you'll need to define and create a simple metric.
:::
-->

## シンプルメトリクスの例

```yaml
  metrics: 
    - name: customers
      description: Count of customers
      type: simple # Pointers to a measure you created in a semantic model
      label: Count of customers
      type_params:
        measure: 
          name: customers # The measure you are creating a proxy of.
          fill_nulls_with: 0 
          join_to_timespine: true
          alias: customer_count
          filter: {{ Dimension('customer__customer_total') }} >= 20
    - name: large_orders
      description: "Order with order values over 20."
      type: simple
      label: Large orders
      type_params:
        measure: 
          name: orders
      filter: | # For any metric you can optionally include a filter on dimension values
        {{Dimension('customer__order_total_dim')}} >= 20
```

## 関連ドキュメント
- [シンプルメトリクス、派生メトリクス、または比率メトリクスのnull値を埋める](/docs/build/fill-nulls-advanced)
