---
title: "dbt ls (list) コマンドについて"
sidebar_label: "ls (list)"
description: "dbt の ls (list) コマンドを使用して dbt プロジェクト内のリソースを一覧表示する方法については、このガイドをお読みください。"
id: "list"
---

`dbt ls` コマンドは、dbt プロジェクト内のリソースを一覧表示します。このコマンドは、[dbt run](/reference/commands/run) で提供されるものと同様のセレクター引数を受け入れます。`dbt list` は `dbt ls` のエイリアスです。`dbt ls` は [接続プロファイル](/docs/core/connect-data-platform/connection-profiles) を読み取り、[`target`](/reference/dbt-jinja-functions/target) 固有のロジックを解決しますが、データベースに接続したりクエリを実行したりすることはありません。

### 使用法

```
dbt ls
     [--resource-type {model,semantic_model,source,seed,snapshot,metric,test,exposure,analysis,default,all}]
     [--select SELECTION_ARG [SELECTION_ARG ...]]
     [--models SELECTOR [SELECTOR ...]]
     [--exclude SELECTOR [SELECTOR ...]]
     [--selector YML_SELECTOR_NAME]
     [--output {json,name,path,selector}]
     [--output-keys KEY_NAME [KEY_NAME]]
```

dbt でリソースを選択する方法の詳細については、[リソース選択構文](/reference/node-selection/syntax) を参照してください。

**引数**:
- `--resource-type`: このフラグは、`dbt ls` コマンドで dbt が返す「リソースタイプ」を制限します。デフォルトでは、分析タイプを除くすべてのリソースタイプが `dbt ls` の結果に含まれます。
- `--select`: このフラグは、`dbt ls` コマンドによって返されるノードをフィルタリングするために使用する、1 つ以上の選択タイプの引数を指定します。
- `--models`: `--select` フラグと同様に、このフラグはノードを選択するために使用されます。これは `--resource-type=model` を意味し、`dbt ls` コマンドの結果にはモデルのみが返されます。後方互換性のためにのみサポートされています。
- `--exclude`: 返されるノードのリストから _除外_ するセレクターを指定します。
- `--selector`: このフラグは、`selectors.yml` ファイルで定義された名前付きセレクターを 1 つ指定します。
- `--output`: このフラグは、`dbt ls` コマンドの出力形式を制御します。
- `--output-keys`: `--output json` の場合、このフラグは出力に含めるノードプロパティを制御します。

`dbt ls` コマンドは、無効化されたモデルや、無効化されたモデルに依存するスキーマテストを出力に含めないことに注意してください。返されるすべてのリソースの `config.enabled` 値は `true` になります。

### 使用例

**パッケージ別にモデルを一覧表示**
```
$ dbt ls --select snowplow.*
snowplow.snowplow_base_events
snowplow.snowplow_base_web_page_context
snowplow.snowplow_id_map
snowplow.snowplow_page_views
snowplow.snowplow_sessions
...
```

**タグ名によるテストの一覧表示**
```
$ dbt ls --select tag:nightly --resource-type test
my_project.schema_test.not_null_orders_order_id
my_project.schema_test.unique_orders_order_id
my_project.schema_test.not_null_products_product_id
my_project.schema_test.unique_products_product_id
...
```

**増分モデルのスキーマテストの一覧表示**
```
$ dbt ls --select config.materialized:incremental,test_type:schema
model.my_project.logs_parsed
model.my_project.events_categorized
```

**JSON出力の一覧表示**
```
$ dbt ls --select snowplow.* --output json
{"name": "snowplow_events", "resource_type": "model", "package_name": "snowplow",  ...}
{"name": "snowplow_page_views", "resource_type": "model", "package_name": "snowplow",  ...}
...
```

**カスタムキーを使用した JSON 出力の一覧表示**

```
$ dbt ls --select snowplow.* --output json --output-keys "name resource_type description"
{"name": "snowplow_events", "description": "This is a pretty cool model",  ...}
{"name": "snowplow_page_views", "description": "This model is even cooler",  ...}
...
```

**セマンティックモデルの一覧表示**

List all resources upstream of your orders semantic model:
```
dbt ls -s +semantic_model:orders
```

**ファイルパスの一覧表示**
```
dbt ls --select snowplow.* --output path
models/base/snowplow_base_events.sql
models/base/snowplow_base_web_page_context.sql
models/identification/snowplow_id_map.sql
...
```
