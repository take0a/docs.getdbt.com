---
title: メトリックの作成
id: metrics-overview
description: "メトリックは、同じ dbt プロジェクト リポジトリ内のセマンティック モデルと同じまたは別の YAML ファイルで定義できます。"
sidebar_label: "Creating metrics"
tags: [Metrics, Semantic Layer]
pagination_next: "docs/build/cumulative"
---
  
[セマンティックモデル](/docs/build/semantic-models)をビルドしたら、いよいよメトリックの追加作業を開始します。このページでは、dbt プロジェクトに追加できる、サポートされている様々なメトリックの種類について説明します。

メトリックは YAML ファイルで定義する必要があります。セマンティックモデルと同じファイル内、または dbt プロジェクトのサブディレクトリにある別の YAML ファイル内に定義してください。モデルの `config` ブロック内で定義することはできません。

メトリック定義のキーは次のとおりです:

<!-- for v1.8 and higher -->

<VersionBlock firstVersion="1.8">

| Parameter | Description | Required | Type |
| --------- | ----------- | ---- | ---- |
| `name` | メトリックの参照名を入力します。この名前は一意のメトリック名である必要があり、小文字、数字、アンダースコアを使用できます。  | Required | String |
| `description` | 指標を説明してください。 | Optional | String |
| `type` | メトリックのタイプを定義します。これは、「変換」、「累積」、「派生」、「比率」、または「単純」のいずれかになります。 | Required | String |
| `type_params` | メトリックを構成するために使用される追加のパラメータ。`type_params` はメトリック タイプごとに異なります。 | Required | Dict |
| `label` | 下流ツールでの表示値を定義する必須文字列。プレーンテキスト、スペース、引用符（例：`orders_total`、`"orders_total"`）が使用できます。  | Required | String |
| `config` | メトリックの設定を指定するには、[`config`](/reference/resource-properties/config) プロパティを使用します。[`meta`](/reference/resource-configs/meta)、[`group`](/reference/resource-configs/group)、[`enabled`](/reference/resource-configs/enabled) 設定がサポートされています。 | Optional | Dict |
| `filter` | オプションで、任意の指標タイプに[filter](#filters)文字列を追加して、指標計算中にディメンション、エンティティ、時間ディメンション、その他の指標にフィルターを適用できます。WHERE句のように考えてください。 | Optional | String |

メトリック仕様の構成の完全な例を次に示します:

<File name="models/metrics/file_name.yml" >

```yaml
metrics:
  - name: metric name                     ## Required
    description: description               ## Optional
    type: the type of the metric          ## Required
    type_params:                          ## Required
      - specific properties for the metric type
    config:                               ## Optional
      meta:
        my_meta_config:  'config'         ## Optional
    label: The display name for your metric. This value will be shown in downstream tools. ## Required
    filter: |                             ## Optional            
      {{  Dimension('entity__name') }} > 0 and {{ Dimension(' entity__another_name') }} is not
      null and {{ Metric('metric_name', group_by=['entity_name']) }} > 5
```

</File>
</VersionBlock>

<!-- for v1.7 and lower -->

<VersionBlock lastVersion="1.7">

| Parameter | Description | Required | Type  |
| --------- | ----------- | ---- | ---- |
| `name` | Provide the reference name for the metric. This name must be unique amongst all metrics.   | Required | String |
| `description` | Describe your metric.   | Optional | String |
| `type` | Define the type of metric, which can be `simple`, `ratio`, `cumulative`, or `derived`.  | Required | String |
| `type_params` | Additional parameters used to configure metrics. `type_params` are different for each metric type. | Required | Dict |
| `config` | Provide the specific configurations for your metric.   | Optional | Dict |
| `meta` | Use the [`meta` config](/reference/resource-configs/meta) to set metadata for a resource.  | Optional | String |
| `label` | Required string that defines the display value in downstream tools. Accepts plain text, spaces, and quotes (such as `orders_total` or `"orders_total"`).   | Required | String |
| `filter` | You can optionally add a filter string to any metric type, applying filters to dimensions, entities, or time dimensions during metric computation. Consider it as your WHERE clause.   | Optional | String |

Here's a complete example of the metrics spec configuration:

<File name="models/metrics/file_name.yml" >

```yaml
metrics:
  - name: metric name                     ## Required
    description: same as always           ## Optional
    type: the type of the metric          ## Required
    type_params:                          ## Required
      - specific properties for the metric type
    config: here for `enabled`            ## Optional
    meta:
        my_meta_direct: 'direct'           ## Optional
    label: The display name for your metric. This value will be shown in downstream tools. ## Required
    filter: |                             ## Optional            
      {{  Dimension('entity__name') }} > 0 and {{ Dimension(' entity__another_name') }} is not
      null and {{ Metric('metric_name', group_by=['entity_name']) }} > 5
```
</File>

</VersionBlock>

import SLCourses from '/snippets/_sl-course.md';

<SLCourses/>

## 指標のデフォルトの粒度

<VersionBlock lastVersion="1.8">
Default time granularity for metrics is useful if your time dimension has a very fine grain, like second or hour, but you typically query metrics rolled up at a coarser grain. 

Default time granularity for metrics is available now in [the "Latest" release track in dbt Cloud](/docs/dbt-versions/cloud-release-tracks), and it will be available in [dbt Core v1.9+](/docs/dbt-versions/core-upgrade/upgrading-to-v1.9). 


</VersionBlock>

<VersionBlock firstVersion="1.9">

メトリクスのデフォルトの時間粒度を、デフォルトの集計時間ディメンション（`metric_time`）の粒度と異なる粒度で定義できます。これは、時間ディメンションの粒度が秒や時間など非常に細かいものの、通常はより粗い粒度で集計されたメトリクスをクエリする場合に便利です。

粒度はメトリクスの `time_granularity` パラメータを使用して設定でき、デフォルトは `day` です。ディメンションの粒度が粗いために day が使用できない場合は、ディメンションに定義されている粒度がデフォルトになります。

### 例

- 「orders」というセマンティックモデルがあり、「order_time」という時間ディメンションがあるとします。
- 「orders」指標をデフォルトで「monthly」にロールアップしたいとします。ただし、これらの指標を時間単位で表示するオプションも必要です。
- 「order_time」ディメンションの「time_granularity」パラメータを「hour」に設定し、指標の「time_granularity」パラメータを「month」に設定します。

```yaml
semantic_models:
  ...
  dimensions:
    - name: order_time
      type: time
      type_params:
      time_granularity: hour
  measures:
    - name: orders
      expr: 1
      agg: sum

metrics:
  - name: orders
    type: simple
    label: Count of Orders
    type_params:
      measure:
        name: orders
    time_granularity: month -- Optional, defaults to day
```

メトリクスはセマンティックモデルと同じYAMLファイルで定義できますが、`semantic_models`キー内にネストせず、独立したトップレベルセクションとして定義する必要があります。または、同じdbtプロジェクトリポジトリ内の任意のサブディレクトリにある専用のYAMLファイルでメトリクスを定義することもできます。

</VersionBlock>

## コンバージョン指標

[コンバージョン指標](/docs/build/conversion) は、設定された期間内にエンティティのベースイベントとそれに続くコンバージョンイベントがいつ発生したかを追跡するのに役立ちます。

<File name="models/metrics/file_name.yml" >

```yaml
metrics:
  - name: The metric name 
    description: The metric description 
    type: conversion 
    label: YOUR_LABEL 
    type_params: #
      conversion_type_params: 
        entity: ENTITY
        calculation: CALCULATION_TYPE 
        base_measure: 
          name: The name of the measure 
          fill_nulls_with: Set the value in your metric definition instead of null (such as zero) 
          join_to_timespine: true/false
        conversion_measure:
          name: The name of the measure 
          fill_nulls_with: Set the value in your metric definition instead of null (such as zero) 
          join_to_timespine: true/false
        window: TIME_WINDOW
        constant_properties:
          - base_property: DIMENSION or ENTITY 
            conversion_property: DIMENSION or ENTITY 
```
</File>

## 累積メトリクス

[累積メトリクス](/docs/build/cumulative)は、指定された期間の測定値を集計します。期間を指定しない場合は、記録された期間全体にわたって測定値が累積されます。累積メトリクスを追加する前に、[タイムスパインモデル](/docs/build/metricflow-time-spine)を作成する必要があることに注意してください。

<File name="models/metrics/file_name.yml" >

```yaml
# Cumulative metrics aggregate a measure over a given window. The window is considered infinite if no window parameter is passed (accumulate the measure over all of time)
metrics:
  - name: wau_rolling_7
    type: cumulative
    label: Weekly active users
    type_params:
      measure:
        name: active_users
        fill_nulls_with: 0
        join_to_timespine: true
      cumulative_type_params:
        window: 7 days
```
</File>

## 派生メトリクス

[派生メトリクス](/docs/build/derived)は、他のメトリクスの式として定義されます。派生メトリクスを使用すると、メトリクスに基づいて計算を行うことができます。

<File name="models/metrics/file_name.yml" >

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
```
</File>

<!-- not supported
### Expression metrics
Use [expression metrics](/docs/build/expr) when you're building a metric that involves a SQL expression of multiple measures.

```yaml
# Expression metric
metrics:
  name: revenue_usd
  type: expr # Expression metrics allow you to pass in any valid SQL expression.
  type_params:
    expr: transaction_amount_usd - cancellations_usd + alterations_usd # Define the SQL expression 
    measures: # Define all the measures that are to be used in this expression metric 
      - transaction_amount_usd
      - cancellations_usd
      - alterations_usd
```
-->

## 比率メトリクス

[比率メトリクス](/docs/build/ratio)は、分子メトリクスと分母メトリクスで構成されます。`filter`文字列は、分子と分母の両方に適用することも、分子または分母のいずれかに個別に適用することもできます。

<File name="models/metrics/file_name.yml" >

```yaml
metrics:
  - name: cancellation_rate
    type: ratio
    label: Cancellation rate
    type_params:
      numerator: cancellations
      denominator: transaction_amount
    filter: |   
      {{ Dimension('customer__country') }} = 'MX'
  - name: enterprise_cancellation_rate
    type: ratio
    type_params:
      numerator:
        name: cancellations
        filter: {{ Dimension('company__tier') }} = 'enterprise'  
      denominator: transaction_amount
    filter: | 
      {{ Dimension('customer__country') }} = 'MX' 
```
</File>

## シンプルメトリクス

[シンプルメトリクス](/docs/build/simple) は、メジャーを直接参照します。これは、1つのメジャーのみを入力として受け取る関数と考えることができます。

- `name` - このパラメータを使用して、メトリクスの参照名を定義します。名前はメトリクス間で一意である必要があり、小文字、数字、アンダースコアを含めることができます。この名前を使用して、dbt セマンティックレイヤー API からメトリクスを呼び出すことができます。

**注:** `create_metric: True` パラメータを使用してすでにメトリクスを定義している場合は、シンプルメトリクスを作成する必要はありません。ただし、メトリクスに制約を追加する場合は、シンプルタイプのメトリクスを作成する必要があります。

<File name="models/metrics/file_name.yml" >

```yaml
metrics:
  - name: cancellations
    description: The number of cancellations
    type: simple
    label: Cancellations
    type_params:
      measure:
        name: cancellations_usd  # Specify the measure you are creating a proxy for.
        fill_nulls_with: 0
        join_to_timespine: true
    filter: |
      {{ Dimension('order__value')}} > 100 and {{Dimension('user__acquisition')}} is not null
```
</File>

## フィルター

フィルターはJinjaテンプレートを使用して設定されます。フィルター内のエンティティ、ディメンション、時間ディメンション、または指標を参照するには、次の構文を使用します。

指標フィルターで指標をディメンションとして使用する方法の詳細については、[ディメンションとしての指標](/docs/build/ref-metrics-in-filters)を参照してください。

<VersionBlock firstVersion="1.8">

<File name="models/metrics/file_name.yml" >

```yaml
filter: | 
  {{ Entity('entity_name') }}

filter: |  
  {{ Dimension('primary_entity__dimension_name') }}

filter: |  
  {{ TimeDimension('time_dimension', 'granularity') }}

filter: |  
 {{ Metric('metric_name', group_by=['entity_name']) }}  

```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.7">


<File name="models/metrics/file_name.yml" >

```yaml
filter: | 
  {{ Entity('entity_name') }}

filter: |  
  {{ Dimension('primary_entity__dimension_name') }}

filter: |  
  {{ TimeDimension('time_dimension', 'granularity') }}

```
</File>
</VersionBlock>

たとえば、月ごとにグループ化された注文日ディメンションをフィルタリングする場合は、次の構文を使用します:

```yaml
filter: |  
  {{ TimeDimension('order_date', 'month') }}

```

## その他の設定

メトリクスにメタデータを追加設定できます。これらのメタデータは、後から他のツールで使用できます。メタデータの使用方法は、連携パートナーによって異なります。

- **説明** - メトリクスの詳細な説明を記入してください。

## 関連ドキュメント

- [セマンティックモデル](/docs/build/semantic-models)
- [指標のnull値を埋める](/docs/build/fill-nulls-advanced)
- [指標フィルターで指標をディメンションとして扱う](/docs/build/ref-metrics-in-filters)
