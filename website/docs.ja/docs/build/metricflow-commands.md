---
title: MetricFlow コマンド
id: metricflow-commands
description: "MetricFlow コマンドを使用して、dbt プロジェクト内のメトリックとメタデータをクエリします。"
sidebar_label: "MetricFlow commands"
tags: [Metrics, Semantic Layer]
---

dbt プロジェクトでメトリクスを定義すると、MetricFlow コマンドを使用してメトリクス、ディメンション、ディメンション値のクエリを実行し、構成を検証できるようになります。

MetricFlow を使用すると、[dbt Cloud](/docs/cloud/about-develop-dbt) または [dbt Core](/docs/core/installation-overview) で dbt プロジェクト内のメトリクスを定義およびクエリできます。
ユニバーサルな [dbt Semantic Layer](/docs/use-dbt-semantic-layer/dbt-sl) のパワーを活用し、下流ツールでそれらのメトリクスを動的にクエリするには、dbt Cloud [Team または Enterprise](https://www.getdbt.com/pricing/) アカウントが必要です。

MetricFlow は、Python バージョン 3.8、3.9、3.10、3.11 と互換性があります。

## MetricFlow

MetricFlow は、dbt プロジェクトでメトリクスの定義とクエリを実行できる dbt パッケージです。
MetricFlow を使用すると、dbt Cloud CLI、dbt Cloud IDE、または dbt Core で dbt プロジェクト内のメトリクスをクエリできます。

MetricFlow を dbt Cloud と併用すると、バージョン管理が不要になります。dbt Cloud アカウントが自動的にバージョン管理を行います。

dbt Cloud ジョブは、`dbt sl validate` コマンドをサポートしており、[セマンティックノードを自動的にテスト](/docs/deploy/ci-jobs#semantic-validations-in-ci) します。
また、MetricFlow をインストール (`python -m pip install metricflow`) することで、git プロバイダー (GitHub Actions など) で MetricFlow 検証を追加することもできます。
これにより、PR の継続的インテグレーション チェックの一環として MetricFlow コマンドを実行できます。

<Tabs>

<TabItem value="cloud" label="MetricFlow with dbt Cloud">

dbt Cloud では、[dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) または [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) で MetricFlow コマンドを直接実行できます。

dbt Cloud CLI ユーザーの場合、MetricFlow コマンドは dbt Cloud CLI に組み込まれているため、dbt Cloud CLI をインストールするとすぐに実行でき、MetricFlow を別途インストールする必要はありません。
dbt Cloud アカウントが自動的にバージョン管理を行うため、バージョン管理は不要です。

</TabItem>

<TabItem value="core" label="MetricFlow with dbt Core">  

[MetricFlow](https://github.com/dbt-labs/metricflow#getting-started)は[PyPI](https://pypi.org/project/dbt-metricflow/)からインストールできます。WindowsまたはLinuxオペレーティングシステムにMetricFlowをインストールするには、`pip`を使用する必要があります:

<VersionBlock lastVersion="1.7">
 
1. Create or activate your virtual environment `python -m venv venv`
2. Run `pip install dbt-metricflow`
  * You can install MetricFlow using PyPI as an extension of your dbt adapter in the command line. To install the adapter, run `python -m pip install "dbt-metricflow[your_adapter_name]"` and add the adapter name at the end of the command. For example, for a Snowflake adapter run `python -m pip install "dbt-metricflow[snowflake]"`

</VersionBlock>

<VersionBlock firstVersion="1.8">
 
1. 仮想環境を作成またはアクティブ化します `python -m venv venv`
2. `pip install dbt-metricflow` を実行します
* コマンドラインで、PyPI を使用して MetricFlow を dbt アダプタの拡張機能としてインストールできます。アダプタをインストールするには、`python -m pip install "dbt-metricflow[adapter_package_name]"` を実行し、コマンドの末尾にアダプタ名を追加します。例えば、Snowflake アダプタの場合は、`python -m pip install "dbt-metricflow[dbt-snowflake]"` を実行します。

</VersionBlock>

**注意**: dbt Core、アダプタ、MetricFlow 間のバージョン管理が必要になります。

Metafont LaTeX パッケージがインストールされている場合、MetricFlow の `mf` コマンドはエラーを返しますのでご注意ください。`mf` コマンドを実行するには、パッケージをアンインストールしてください。

</TabItem>
</Tabs>

## MetricFlow コマンド

MetricFlow は、メタデータの取得とメトリクスのクエリを実行するための次のコマンドを提供します。

<Tabs>
<TabItem value="cloudcommands" label="Commands for dbt Cloud">

コマンド名の前に「dbt sl」プレフィックスを付けると、dbt Cloud IDE または dbt Cloud CLI で実行できます。たとえば、すべてのメトリックを一覧表示するには、「dbt sl list metrics」を実行します。

dbt Cloud CLI ユーザーは、ターミナルで「dbt sl --help」を実行すると、MetricFlow のコマンドとフラグの完全な一覧が表示されます。

次の表は、dbt Cloud IDE および dbt Cloud CLI と互換性のあるコマンドの一覧です:

| <div style={{width:'250px'}}>Command</div>  | <div style={{width:'100px'}}>Description</div> | dbt Cloud IDE | dbt Cloud CLI |
|---------|-------------|---------------|---------------|
| [`list metrics`](#list-metrics) | ディメンションを含むメトリックを一覧表示します。 |  ✅ | ✅ |
| [`list dimensions`](#list) | メトリックの一意のディメンションを一覧表示します。 |  ✅  | ✅ |
| [`list dimension-values`](#list-dimension-values) | メトリックを含むディメンションをリストします。 | ✅ | ✅ |
| [`list entities`](#list-entities) | すべての一意のエンティティを一覧表示します。  |  ✅  | ✅ |
| [`list saved-queries`](#list-saved-queries) | 利用可能な保存済みクエリを一覧表示します。保存済みクエリの下にリストされている各エクスポートを表示するには `--show-exports` フラグを使用し、各保存済みクエリで使用される完全なクエリパラメータを表示するには `--show-parameters` フラグを使用します。 |  ✅ | ✅ |
| [`query`](#query) | コマンドラインインターフェースで表示するメトリクス、保存済みクエリ、ディメンションをクエリします。メトリクスとディメンションのクエリを実行するには、[クエリ例](#query-examples)を参照してください（メトリクスのクエリ、`where` フィルターの使用、`order` の追加など）。  |  ✅ | ✅ |
| [`validate`](#validate) | セマンティック モデルの構成を検証します。 |  ✅ | ✅ |
| [`export`](#export) |  開発環境でのテストとエクスポート生成のために、保存済みの単一のクエリに対してエクスポートを実行します。また、`--select` フラグを使用して、保存済みのクエリから特定のエクスポートを指定することもできます。 |  ❌ | ✅ |
| [`export-all`](#export-all) | 保存された複数のクエリのエクスポートを一度に実行し、時間と労力を節約します。 |  ❌ | ✅ |


<!--below commands aren't supported in dbt cloud yet
- [`health-checks`](#health-checks) &mdash; Performs data platform health check.
- [`tutorial`](#tutorial) &mdash; Dedicated MetricFlow tutorial to help get you started.
-->

:::tip メトリックの変更を反映するために dbt parse を実行します
指標に変更を加える際は、少なくとも `dbt parse` を実行して dbt セマンティックレイヤーを更新してください。
これにより `semantic_manifest.json` ファイルが更新され、指標のクエリ時に変更が反映されます。
`dbt parse` を実行することで、すべてのモデルを再構築する必要がなくなります。
::: 

<Expandable alt_header="dbt Cloud CLI を使用してメトリックをクエリまたはプレビューするにはどうすればよいですか?">

dbt Cloud CLI を使用してメトリックをクエリまたはプレビューする方法については、次の短いビデオ デモをご覧ください:

<LoomVideo id='09e2b287f063497d888f4bed91469d79' />

</Expandable>

</TabItem>

<TabItem value="corecommands" label="dbt Core のコマンド">

dbt Core で実行するには、コマンド名の前に `mf` プレフィックスを付けます。
たとえば、すべてのメトリックを一覧表示するには、`mf list metrics` を実行します。

- [`list metrics`](#list-metrics) &mdash; ディメンションを含むメトリックを一覧表示します。
- [`list dimensions`](#list) &mdash; メトリックの一意のディメンションを一覧表示します。
- [`list dimension-values`](#list-dimension-values) &mdash; メトリックを含むディメンションを一覧表示します。
- [`list entities`](#list-entities) &mdash; すべての一意のエンティティを一覧表示します。
- [`validate-configs`](#validate-configs) &mdash; セマンティック モデル構成を検証します。
- [`health-checks`](#health-checks) &mdash; データ プラットフォームのヘルスチェックを実行します。
- [`tutorial`](#tutorial) &mdash; MetricFlow を使い始める際に役立つ専用のチュートリアルをご用意しています。
- [`query`](#query) - コマンドラインインターフェースで表示するメトリクスとディメンションをクエリします。[クエリの例](#query-examples)を参考に、使い始めてください。
  
</TabItem>
</Tabs>

## 指標の一覧表示
このコマンドは、利用可能なディメンションとともに指標を一覧表示します。

```bash
dbt sl list metrics <metric_name> # In dbt Cloud

mf list metrics <metric_name> # In dbt Core

Options:
  --search TEXT          Filter available metrics by this search term
  --show-all-dimensions  Show all dimensions associated with a metric.
  --help                 Show this message and exit.
```

## ディメンションの一覧表示

このコマンドは、1つまたは複数の指標の固有のディメンションをすべて一覧表示します。
複数の指標をクエリする場合は、共通のディメンションのみを表示します:

```bash
dbt sl list dimensions --metrics <metric_name> # In dbt Cloud

mf list dimensions --metrics <metric_name> # In dbt Core

Options:
  --metrics SEQUENCE  List dimensions by given metrics (intersection). Ex. --metrics bookings,messages
  --help              Show this message and exit.
```

## ディメンション値の一覧表示

このコマンドは、対応する指標を持つすべてのディメンション値を一覧表示します:

```bash
dbt sl list dimension-values --metrics <metric_name> --dimension <dimension_name> # In dbt Cloud

mf list dimension-values --metrics <metric_name> --dimension <dimension_name> # In dbt Core

Options:
  --dimension TEXT    Dimension to query values from  [required]
  --metrics SEQUENCE  Metrics that are associated with the dimension
                      [required]
  --end-time TEXT     Optional iso8601 timestamp to constraint the end time of
                      the data (inclusive)
                      *Not available in dbt Cloud yet
  --start-time TEXT   Optional iso8601 timestamp to constraint the start time
                      of the data (inclusive)
                      *Not available in dbt Cloud yet
  --help              Show this message and exit.
```

## エンティティの一覧表示

このコマンドは、すべての一意のエンティティを一覧表示します:

```bash
dbt sl list entities --metrics <metric_name> # In dbt Cloud 

mf list entities --metrics <metric_name> # In dbt Core

Options:
  --metrics SEQUENCE  List entities by given metrics (intersection). Ex. --metrics bookings,messages
  --help              Show this message and exit.
```

## 保存済みクエリの一覧表示

このコマンドは、利用可能なすべての保存済みクエリを一覧表示します:

```bash
dbt sl list saved-queries
```

保存されたクエリの下にリストされている各エクスポートを表示するには、`--show-exports` フラグ (またはオプション) を追加することもできます:

```bash
dbt sl list saved-queries --show-exports
```

**Output**

```bash
dbt sl list saved-queries --show-exports

The list of available saved queries:
- new_customer_orders
  exports:
       - Export(new_customer_orders_table, exportAs=TABLE)
       - Export(new_customer_orders_view, exportAs=VIEW)
       - Export(new_customer_orders, alias=orders, schemas=customer_schema, exportAs=TABLE)
```

## 検証

次のコマンドは、定義されたセマンティックモデル構成に対して検証を実行します。

```bash
dbt sl validate # For dbt Cloud users
mf validate-configs # For dbt Core users

Options:
  --timeout                       # dbt Cloud only
                                  Optional timeout for data warehouse validation in dbt Cloud.
  --dw-timeout INTEGER            # dbt Core only
                                  Optional timeout for data warehouse
                                  validation steps. Default None.
  --skip-dw                       # dbt Core only
                                  Skips the data warehouse validations.
  --show-all                      # dbt Core only
                                  Prints warnings and future errors.
  --verbose-issues                # dbt Core only
                                  Prints extra details about issues.
  --semantic-validation-workers INTEGER  # dbt Core only
                                  Uses specified number of workers for large configs.
  --help                          Show this message and exit.
```

## ヘルスチェック

次のコマンドは、構成ファイルで指定したデータプラットフォームに対してヘルスチェックを実行します。

dbt Cloud では、ヘルスチェックの実行に dbt Cloud の認証情報を使用するため、`health-checks` コマンドは必要ありません。

```bash
mf health-checks # In dbt Core
```

## チュートリアル

MetricFlow を使い始めるには、専用のチュートリアルをご覧ください:
<!--dbt sl tutorial # In dbt Cloud-->

```bash
mf tutorial # In dbt Core
```

## クエリ

MetricFlow で新しいクエリを作成し、データプラットフォームに対して実行します。
クエリは次の結果を返します:

```bash
dbt sl query --metrics <metric_name> --group-by <dimension_name> # In dbt Cloud 
dbt sl query --saved-query <name> # In dbt Cloud

mf query --metrics <metric_name> --group-by <dimension_name> # In dbt Core

Options:

  --metrics SEQUENCE       Syntax to query single metrics: --metrics metric_name
                           For example, --metrics bookings
                           To query multiple metrics, use --metrics followed by the metric names, separated by commas without spaces.
                           For example,  --metrics bookings,messages

  --group-by SEQUENCE      Syntax to group by single dimension/entity: --group-by dimension_name
                           For example, --group-by ds
                           For multiple dimensions/entities, use --group-by followed by the dimension/entity names, separated by commas without spaces.
                           For example, --group-by ds,org
                           

  --end-time TEXT          Optional iso8601 timestamp to constraint the end
                           time of the data (inclusive).
                           *Not available in dbt Cloud yet 

  --start-time TEXT        Optional iso8601 timestamp to constraint the start
                           time of the data (inclusive)
                           *Not available in dbt Cloud yet

  --where TEXT             SQL-like where statement provided as a string and wrapped in quotes.
                           All filter items must explicitly reference fields or dimensions that are part of your model.
                           To query a single statement: ---where "{{ Dimension('order_id__revenue') }} > 100"
                           To query multiple statements: --where "{{ Dimension('order_id__revenue') }} > 100 and {{ Dimension('user_count') }} < 1000"
                           To add a dimension filter, use the `Dimension()` template wrapper to indicate that the filter item is part of your model. 
                           Refer to the [FAQ](#faqs) for more info on how to do this using a template wrapper.

  --limit TEXT             Limit the number of rows out using an int or leave
                           blank for no limit. For example: --limit 100

  --order-by SEQUENCE     Specify metrics, dimension, or group bys to order by.
                          Add the `-` prefix to sort query in descending (DESC) order. 
                          Leave blank for ascending (ASC) order.
                          For example, to sort metric_time in DESC order: --order-by -metric_time 
                          To sort metric_time in ASC order and revenue in DESC order:  --order-by metric_time,-revenue

  --csv FILENAME           Provide filepath for data frame output to csv

 --compile (dbt Cloud)    In the query output, show the query that was
 --explain (dbt Core)     executed against the data warehouse         
                           

  --show-dataflow-plan     Display dataflow plan in explain output

  --display-plans          Display plans (such as metric dataflow) in the browser

  --decimals INTEGER       Choose the number of decimal places to round for
                           the numerical values

  --show-sql-descriptions  Shows inline descriptions of nodes in displayed SQL

  --help                   Show this message and exit.
  ```


## クエリ例

このセクションでは、指標とディメンションのクエリに使用できる様々なタイプのクエリ例を紹介します。
クエリ例は以下のとおりです。

- [指標のクエリ](#query-metrics)
- [ディメンションのクエリ](#query-dimensions)
- [`order`/`limit` 関数の追加](#add-orderlimit)
- [`where` 句の追加](#add-where-clause)
- [時間によるフィルタリング](#filter-by-time)
- [保存済みクエリのクエリ](#query-saved-queries)

### 指標のクエリ {#query-metrics}

この例を使用して、ディメンション別に複数のメトリックをクエリし、`metric_time` 別に `order_total` および `users_active` メトリックを返します。

**Query**
```bash
dbt sl query --metrics order_total,users_active --group-by metric_time # In dbt Cloud

mf query --metrics order_total,users_active --group-by metric_time # In dbt Core
```

**Result**
```bash
✔ Success 🦄 - query completed after 1.24 seconds
| METRIC_TIME   |   ORDER_TOTAL |
|:--------------|---------------:|
| 2017-06-16    |         792.17 |
| 2017-06-17    |         458.35 |
| 2017-06-18    |         490.69 |
| 2017-06-19    |         749.09 |
| 2017-06-20    |         712.51 |
| 2017-06-21    |         541.65 |
```

### ディメンションのクエリ {#query-dimensions}

クエリには複数のディメンションを含めることができます。
例えば、注文が食品かどうかを確認するために、「is_food_order」ディメンションでグループ化することができます。
ディメンションをクエリする際は、そのディメンションのプライマリエンティティを指定する必要があります。
次の例では、プライマリエンティティは「order_id」です。

**Query**
```bash
dbt sl query --metrics order_total --group-by order_id__is_food_order # In dbt Cloud

mf query --metrics order_total --group-by order_id__is_food_order # In dbt Core
```

**Result**
```bash
 Success 🦄 - query completed after 1.70 seconds
| METRIC_TIME   | IS_FOOD_ORDER   |   ORDER_TOTAL |
|:--------------|:----------------|---------------:|
| 2017-06-16    | True            |         499.27 |
| 2017-06-16    | False           |         292.90 |
| 2017-06-17    | True            |         431.24 |
| 2017-06-17    | False           |          27.11 |
| 2017-06-18    | True            |         466.45 |
| 2017-06-18    | False           |          24.24 |
| 2017-06-19    | False           |         300.98 |
| 2017-06-19    | True            |         448.11 |
```

### order/limit の追加 {#add-orderlimit}

order 関数と limit 関数を追加することで、データをフィルタリングし、読みやすい形式で表示できます。
次のクエリは、データセットを10件のレコードに制限し、`metric_time` の降順で並べ替えます。
`-` プレフィックスを使用すると、クエリは降順で並べ替えられることに注意してください。
`-` プレフィックスを使用しないと、クエリは昇順で並べ替えられます。

ディメンションをクエリする場合は、そのディメンションのプライマリエンティティを指定する必要があります。
次の例では、プライマリエンティティは `order_id` です。

**Query**
```bash
# In dbt Cloud 
dbt sl query --metrics order_total --group-by order_id__is_food_order --limit 10 --order-by -metric_time 

# In dbt Core
mf query --metrics order_total --group-by order_id__is_food_order --limit 10 --order-by -metric_time 
```

**Result**
```bash
✔ Success 🦄 - query completed after 1.41 seconds
| METRIC_TIME   | IS_FOOD_ORDER   |   ORDER_TOTAL |
|:--------------|:----------------|---------------:|
| 2017-08-31    | True            |         459.90 |
| 2017-08-31    | False           |         327.08 |
| 2017-08-30    | False           |         348.90 |
| 2017-08-30    | True            |         448.18 |
| 2017-08-29    | True            |         479.94 |
| 2017-08-29    | False           |         333.65 |
| 2017-08-28    | False           |         334.73 |
```

### where 句の追加 {#add-where-clause}

クエリに `where` 句を追加することで、データセットをさらにフィルタリングできます。
次の例は、複数の `where` ステートメントを使用して `is_food_order` でグループ化された `order_total` 指標（食品注文と 2024 年 2 月 1 日以降の週の注文）をクエリする方法を示しています。

**Query**
```bash
# In dbt Cloud 
dbt sl query --metrics order_total --group-by order_id__is_food_order --where "{{ Dimension('order_id__is_food_order') }} = True and {{ TimeDimension('metric_time', 'week') }} >= '2024-02-01'"

# In dbt Core
mf query --metrics order_total --group-by order_id__is_food_order --where "{{ Dimension('order_id__is_food_order') }} = True and TimeDimension('metric_time', 'week') }} >= '2024-02-01'"
```

注:
- ディメンションの種類によって使用する構文が異なります。日付フィールドがある場合は、`Dimension` ではなく `TimeDimension` を使用してください。
- ディメンションをクエリする場合は、そのディメンションのプライマリエンティティを指定する必要があります。先ほど共有した例では、プライマリエンティティは `order_id` です。

**Result**
```bash
 ✔ Success 🦄 - query completed after 1.06 seconds
| METRIC_TIME   | IS_FOOD_ORDER   |   ORDER_TOTAL |
|:--------------|:----------------|---------------:|
| 2017-08-31    | True            |         459.90 |
| 2017-08-30    | True            |         448.18 |
| 2017-08-29    | True            |         479.94 |
| 2017-08-28    | True            |         513.48 |
| 2017-08-27    | True            |         568.92 |
| 2017-08-26    | True            |         471.95 |
| 2017-08-25    | True            |         452.93 |
| 2017-08-24    | True            |         384.40 |
| 2017-08-23    | True            |         423.61 |
| 2017-08-22    | True            |         401.91 |
```

### 時間でフィルタリング {#filter-by-time}

時間でフィルタリングするには、専用の開始時間と終了時間のオプションがあります。
これらのオプションを使用して時間でフィルタリングすると、MetricFlowは適切な場合にwhereフィルターを押し下げることで、クエリのパフォーマンスをさらに最適化できます。

ディメンションをクエリする場合は、そのディメンションのプライマリエンティティを指定する必要があります。
次の例では、プライマリエンティティは「order_id」です。
<!--
bash not support in cloud yet
# In dbt Cloud
dbt sl query --metrics order_total --group-by order_id__is_food_order --limit 10 --order-by -metric_time --where "is_food_order = True" --start-time '2017-08-22' --end-time '2017-08-27' 
-->
**Query**
```bash
# In dbt Core
mf query --metrics order_total --group-by order_id__is_food_order --limit 10 --order-by -metric_time --where "is_food_order = True" --start-time '2017-08-22' --end-time '2017-08-27' 
```

 **Result**
```bash
✔ Success 🦄 - query completed after 1.53 seconds
| METRIC_TIME   | IS_FOOD_ORDER   |   ORDER_TOTAL |
|:--------------|:----------------|---------------:|
| 2017-08-27    | True            |         568.92 |
| 2017-08-26    | True            |         471.95 |
| 2017-08-25    | True            |         452.93 |
| 2017-08-24    | True            |         384.40 |
| 2017-08-23    | True            |         423.61 |
| 2017-08-22    | True            |         401.91 |
```

### 保存済みクエリのクエリ {#query-saved-queries}

頻繁に使用するクエリに使用できます。
`<name>` を [保存したクエリ](/docs/build/saved-queries) の名前に置き換えてください。

**Query**
```bash
dbt sl query --saved-query <name> # In dbt Cloud

mf query --saved-query <name> # In dbt Core
```

たとえば、dbt Cloud を使用していて、`new_customer_orders` という名前の保存済みクエリがある場合は、`dbt sl query --saved-query new_customer_orders` を実行します。

:::info 保存済みクエリのクエリに関する注意事項
[保存済みクエリ](/docs/build/saved-queries) をクエリする際は、`where`、`limit`、`order`、`compile` などのパラメータを使用できます。
ただし、このコンテキストでは `metric` または `group_by` パラメータにはアクセスできないことに注意してください。
これは、これらのパラメータが保存済みクエリに対して事前に決定された固定パラメータであり、クエリ実行時に変更できないためです。
さらに多くの指標やディメンションをクエリする場合は、標準形式を使用してクエリを作成できます。
:::

## 追加のクエリ例

以下のタブでは、CSVへのエクスポートなど、追加のクエリ例をご覧いただけます。
ニーズに最適なタブを選択してください。

<Tabs>

<TabItem value="eg6" label="--compile/--explain flag">

MetricFlow によって生成された SQL を表示するには、クエリに `--compile` (または dbt Core ユーザーの場合は `--explain`) を追加します。

**Query**

```bash
# In dbt Cloud
dbt sl query --metrics order_total --group-by metric_time,is_food_order --limit 10 --order-by -metric_time --where "is_food_order = True" --start-time '2017-08-22' --end-time '2017-08-27' --compile

# In dbt Core
mf query --metrics order_total --group-by metric_time,is_food_order --limit 10 --order-by -metric_time --where "is_food_order = True" --start-time '2017-08-22' --end-time '2017-08-27' --explain
```

 **Result**
 ```bash
 ✔ Success 🦄 - query completed after 0.28 seconds
🔎 SQL (remove --compile to see data or add --show-dataflow-plan to see the generated dataflow plan):
select
  metric_time
  , is_food_order
  , sum(order_cost) as order_total
from (
  select
    cast(ordered_at as date) as metric_time
    , is_food_order
    , order_cost
  from analytics.js_dbt_sl_demo.orders orders_src_1
  where cast(ordered_at as date) between cast('2017-08-22' as timestamp) and cast('2017-08-27' as timestamp)
) subq_3
where is_food_order = True
group by
  metric_time
  , is_food_order
order by metric_time desc
limit 10
```

</TabItem>

<TabItem value="eg7" label=" Export to CSV">
 
クエリの結果を csv にエクスポートするには、`--csv file_name.csv` フラグを追加します。
`--csv` フラグは dbt Core でのみ使用でき、dbt Cloud ではサポートされていません。

**Query**

```bash

# In dbt Core
mf query --metrics order_total --group-by metric_time,is_food_order --limit 10 --order-by -metric_time --where "is_food_order = True" --start-time '2017-08-22' --end-time '2017-08-27' --csv query_example.csv
```

**Result**
```bash
✔ Success 🦄 - query completed after 0.83 seconds
🖨 Successfully written query output to query_example.csv
```

</TabItem>
</Tabs>

## 時間粒度

オプションで、グローバル時間ディメンション「metric_time」に2つのアンダースコアと粒度単位を追加することで、データを集計する時間粒度を指定できます。
粒度は、「日」、「週」、「月」、「四半期」、「年」でグループ化できます。

以下は、月単位でメトリックデータをクエリする例です。

```bash
dbt sl query --metrics revenue --group-by metric_time__month # In dbt Cloud

mf query --metrics revenue --group-by metric_time__month # In dbt Core
```

## エクスポート

[特定の保存済みクエリのエクスポート](/docs/use-dbt-semantic-layer/exports#exports-for-single-saved-query)を実行します。
このコマンドを使用して、開発環境でエクスポートをテストおよび生成します。
また、`--select`フラグを使用して、保存済みクエリから特定のエクスポートを指定することもできます。
詳細については、[開発中のエクスポート](/docs/use-dbt-semantic-layer/exports#exports-in-development)を参照してください。

エクスポートはdbt Cloudで利用できます。

```bash
dbt sl export 
```

## Export-all

[複数の保存済みクエリのエクスポート](/docs/use-dbt-semantic-layer/exports#exports-for-multiple-saved-queries)を一度に実行します。
このコマンドを使用すると、複数のクエリのエクスポートを同時に管理および実行できるため、時間と労力を節約できます。
詳細については、[開発中のエクスポート](/docs/use-dbt-semantic-layer/exports#exports-in-development)を参照してください。

エクスポートはdbt Cloudで利用できます。

```bash
dbt sl export-all 
```


## FAQs

<DetailsToggle alt_header="ディメンション フィルターを Where フィルターに追加するにはどうすればよいですか?">

where フィルターにディメンションフィルターを追加するには、フィルター項目がモデルの一部であることを示し、テンプレートラッパー（`{{Dimension('primary_entity__dimension_name')}}`）を使用する必要があります。

クエリの例は次のとおりです。`dbt sl query --metrics order_total --group-by metric_time --where "{{Dimension('order_id__is_food_order')}} = True"`。

ただし、テンプレートラッパーを使用する前に、フィルターテンプレートが機能するように、ターミナルで中括弧をエスケープするように設定してください。

<details> 
<summary>中括弧をエスケープするためにターミナルを設定するにはどうすればよいでしょうか?</summary>
<code>.zshrc</code> プロファイルで中括弧をエスケープするように設定するには、<code>setopt</code> コマンドを使用して <code>BRACECCL</code> オプションを有効にします。このオプションを有効にすると、シェルは中括弧をリテラルとして扱い、括弧の展開を防止します。設定手順は以下のとおりです。<br />

1. ターミナルを開きます。
2. <code>nano</code>、<code>vim</code>、またはその他のお好みのテキストエディタを使用して <code>.zshrc</code> ファイルを開きます。<code>nano</code> で開くには、以下のコマンドを使用します。

```bash
nano ~/.zshrc
```
3. ファイルに次の行を追加します:

```bash
setopt BRACECCL
```
4. テキストエディタを保存して終了します（`nano` では、Ctrl + O で保存し、Ctrl + X で終了します）。

5. <code>.zshrc</code> ファイルを source して変更を適用します:

```bash
source ~/.zshrc
```

6. これらの変更を加えると、Zsh シェルは中括弧をリテラル文字として扱い、中括弧の展開を行わなくなります。
つまり、意図しない展開を心配することなく中括弧を使用できます。

シェル設定ファイルを変更すると、シェルの動作に影響が出る可能性があることに注意してください。
シェル設定に精通していない場合は、変更を加える前に <code>.zshrc</code> ファイルのバックアップを作成することをお勧めします。
問題や予期しない動作が発生した場合は、バックアップを復元できます。

</details>

</DetailsToggle>

<DetailsToggle alt_header="dbt Cloud CLI でクエリが 100 行に制限されているのはなぜですか?">

dbt Cloud CLI からのクエリ発行におけるデフォルトの「limit」は 100 行です。
dbt Cloud CLI は通常、開発プロセス中に dbt セマンティック レイヤーをクエリするために使用され、本番環境のレポート作成や大規模なデータセットへのアクセスには使用されないため、このデフォルト設定は、不必要に大きなデータセットが返されるのを防ぐためです。
ほとんどのワークフローでは、データのサブセットのみを返す必要があります。

ただし、必要に応じてクエリで「--limit」オプションを設定することで、この制限を変更できます。
たとえば、1,000 行を返すには、「dbt sl list metrics --limit 1000」を実行します。

</DetailsToggle>

<DetailsToggle alt_header="複数のメトリック、group by、または where ステートメントをクエリするにはどうすればよいですか?">

コマンド内で複数の指標、group by、またはwhere文をクエリするには、以下のガイダンスに従ってください。

- 複数の指標とgroup byをクエリするには、`--metrics`または`--group-by`構文に続けて、指標名またはディメンション/エンティティ名をスペースなしでカンマで区切って指定します。
- 複数の指標の例: `dbt sl query --metrics accounts_active,users_active`
- 複数のディメンション/エンティティの例: `dbt sl query --metrics accounts_active,users_active --group-by metric_time__week,accounts__plan_tier`

- 複数のwhere文をクエリするには、`--where`構文を使用し、文全体を引用符で囲みます。
- 複数のwhere文の例: `dbt sl query --metrics accounts_active,users_active --group-by metric_time__week,accounts__plan_tier --where "metric_time__week >= '2024-02-01' and accounts__plan_tier =  'coco'"`

</DetailsToggle>

<DetailsToggle alt_header="クエリを昇順または降順で並べ替えるにはどうすればよいでしょうか?">

指標をクエリする際は、`--order-by` を使用して、並べ替えの基準となる指標またはグループを指定します。
`order_by` オプションは、指標、ディメンション、および group by に適用されます。

クエリを降順 (DESC) で並べ替えるには、`-` プレフィックスを追加します。昇順 (ASC) の場合は空白のままにします。

- たとえば、指標をクエリし、`metric_time` を降順で並べ替えるには、`dbt sl query --metrics order_total --group-by metric_time --order-by -metric_time` を実行します。
`-metric_time` の `-` プレフィックスは、クエリを降順で並べ替えることに注意してください。
- 指標をクエリし、`metric_time` を昇順で、`revenue` を降順で並べ替えるには、`dbt sl query --metrics order_total --order-by metric_time,-revenue` を実行します。
プレフィックスのない `metric_time` は昇順で並べ替えられ、プレフィックス `-` の付いた `-revenue` はクエリを降順で並べ替えることに注意してください。

</DetailsToggle>
