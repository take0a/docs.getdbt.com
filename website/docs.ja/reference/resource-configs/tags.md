---
description: "Configure tags to label and organize your dbt models and resources."
sidebar_label: "tags"
resource_types: all
datatype: string | [string]
---

<Tabs
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Config property', value: 'other-yaml', },
    { label: 'Config block', value: 'config', },
  ]
}>
<TabItem value="project-yaml">

<File name='dbt_project.yml'>

<VersionBlock lastVersion="1.8">

```yml

[models](/reference/model-configs):
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>] # Supports single strings or list of strings

[snapshots](/reference/snapshot-configs):
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>]

[seeds](/reference/seed-configs):
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>]

```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```yml

[models](/reference/model-configs):
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>] # Supports single strings or list of strings

[snapshots](/reference/snapshot-configs):
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>]

[seeds](/reference/seed-configs):
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>]

[saved-queries:](/docs/build/saved-queries)
  [<resource-path>](/reference/resource-configs/resource-path):
    +tags: <string> | [<string>]

```
</VersionBlock>


</File>
</TabItem>

<TabItem value="other-yaml">

<VersionBlock firstVersion="1.9">

以下の例は、YAML ファイルで dbt リソースにタグを追加する方法を示しています。`resource_type` を、必要に応じて `exposures`、`models`、`snapshots`、`seeds`、または `saved_queries` に置き換えてください。

</VersionBlock>

<VersionBlock lastVersion="1.8">

The following examples show how to add tags to dbt resources in YAML files. Replace `resource_type` with `exposures`, `models`, `snapshots`, or `seeds` as appropriate.
</VersionBlock>

<File name='resource_type/properties.yml'>

```yaml
resource_type:
  - name: resource_name
    config:
      tags: <string> | [<string>] # Supports single strings or list of strings
    # Optional: Add the following specific properties for models
    columns:
      - name: column_name
        tags: <string> | [<string>]
        tests:
          test-name:
            config:
              tags: "single-string" # Supports single string 
              tags: ["string-1", "string-2"] # Supports list of strings
```

</File>

`models/` ディレクトリ内のモデルにタグを適用するには、次の例のように `config` プロパティを追加します:

<File name='models/model.yml'>

```yaml
models:
  - name: my_model
    description: A model description
    config:
      tags: ['example_tag']
```

</File>

</TabItem>

<TabItem value="config">

<File name='models/model.sql'>
```sql
{{ config(
    tags="<string>" | ["<string>"]
) }}
```
</File>

</TabItem>

</Tabs>

## 定義

リソースにタグ（またはタグのリスト）を適用します。

これらのタグは、以下のコマンドを実行する際に、[リソース選択構文](/reference/node-selection/syntax)の一部として使用できます。
- `dbt run --select tag:my_tag` &mdash; 特定のタグが付けられたすべてのモデルを実行します。
- `dbt build --select tag:my_tag` &mdash; 特定のタグが付けられたすべてのリソースをビルドします。
- `dbt seed --select tag:my_tag` &mdash; 特定のタグが付けられたすべてのリソースをシードします。
- `dbt snapshot --select tag:my_tag` &mdash; 特定のタグが付けられたすべてのリソースのスナップショットを作成します。
- `dbt test --select tag:my_tag` &mdash; タグ付けされたモデルに関連付けられたすべてのテストを間接的に実行します。

#### `+` 演算子を使用したタグの使用

[`+` 演算子](/reference/node-selection/graph-operators#the-plus-operator) を使用すると、`tag` 選択に上流または下流の依存関係を含めることができます。
- `dbt run --select tag:my_tag+` &mdash; `my_tag` タグが付いたモデルとそのすべての下流の依存関係を実行します。
- `dbt run --select +tag:my_tag` &mdash; `my_tag` タグが付いたモデルとそのすべての上流の依存関係を実行します。
- `dbt run --select +model_name+` &mdash; モデルとその上流の依存関係、および下流の依存関係を実行します。
- `dbt run --select tag:my_tag+ --exclude tag:exclude_tag` &mdash; `my_tag` タグが付いたモデルとその下流の依存関係を実行し、依存関係に関係なく `exclude_tag` タグが付いたモデルを除外します。


:::tip タグに関する使用上の注意

タグを使用する際は、以下の点にご注意ください。

- タグはプロジェクト階層全体にわたって追加されます。
- 一部のリソースタイプ（ソース、エクスポージャーなど）では、最上位レベルでタグを設定する必要があります。

詳しくは、[使用上の注意](#usage-notes) をご覧ください。
:::

## 例

以下の例は、プロジェクト内のリソースにタグを適用する方法を示しています。タグは `dbt_project.yml`、`schema.yml`、または SQL ファイルで設定できます。

### タグを使用してプロジェクトの一部を実行します

`dbt_project.yml` で、単一の値または文字列としてタグを適用します。次の例では、モデルの1つである `jaffle_shop` モデルに `contains_pii` タグが付けられています。

<File name='dbt_project.yml'>

```yml
models:
  jaffle_shop:
    +tags: "contains_pii"

    staging:
      +tags:
        - "hourly"

    marts:
      +tags:
        - "hourly"
        - "published"

    metrics:
      +tags:
        - "daily"
        - "published"

```
</File>


### モデルへのタグの適用

このセクションでは、`dbt_project.yml`、`schema.yml`、およびSQLファイル内のモデルにタグを適用する方法を説明します。

`dbt_project.yml`ファイル内のモデルにタグを適用するには、以下のコードを追加します:

<File name='dbt_project.yml'>

```yaml
models:
  jaffle_shop:
    +tags: finance # jaffle_shop model is tagged with 'finance'.
```

</File>

`models/` ディレクトリの YAML ファイル内のモデルにタグを適用するには、`config` プロパティを使用して以下を追加します:

<File name='models/stg_customers.yml'>

```yaml
models:
  - name: stg_customers
    description: Customer data with basic cleaning and transformation applied, one row per customer.
    config:
      tags: ['santi'] # stg_customers.yml model is tagged with 'santi'.
    columns:
      - name: customer_id
        description: The unique key for each customer.
        data_tests:
          - not_null
          - unique
```

</File>

SQL ファイル内のモデルにタグを適用するには、次のコードを追加します:

<File name='models/staging/stg_payments.sql'>

```sql
{{ config(
    tags=["finance"] # stg_payments.sql model is tagged with 'finance'.
) }}

select ...

```

</File>

次のコマンドを使用して、特定のタグが付いたリソースを実行します (または特定のタグが付いたリソースを除外します):

```shell
# Run all models tagged "daily"
  dbt run --select tag:daily

# Run all models tagged "daily", except those that are tagged hourly
  dbt run --select tag:daily --exclude tag:hourly
```

### Apply tags to seeds

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    utm_mappings:
      +tags: marketing
```

</File>

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    utm_mappings:
      +tags:
        - marketing
        - hourly
```

</File>

### 保存したクエリにタグを適用する

<VersionBlock lastVersion="1.8">

<VersionCallout version="1.9" />

</VersionBlock>


次の例は、`dbt_project.yml` ファイルに保存されたクエリにタグを適用する方法を示しています。保存されたクエリには `order_metrics` タグが付けられます:

<File name='dbt_project.yml'>

```yml
[saved-queries](/docs/build/saved-queries):
  jaffle_shop:
    customer_order_metrics:
      +tags: order_metrics
```

</File>

次に、次のコマンドを使用して、特定のタグを持つリソースを実行します:

```shell
# Run all resources tagged "order_metrics"
  dbt run --select tag:order_metrics
```

2番目の例は、`semantic_model.yml`ファイルに保存されたクエリに複数のタグを適用する方法を示しています。保存されたクエリには、`order_metrics`と`hourly`のタグが付けられます。

<File name='semantic_model.yml'>

```yaml
saved_queries:
  - name: test_saved_query
    description: "{{ doc('saved_query_description') }}"
    label: Test saved query
    config:
      tags: 
        - order_metrics
        - hourly
```
</File>


次のコマンドを使用して、複数のタグを持つリソースを実行します:

```shell
# Run all resources tagged "order_metrics" and "hourly"
  dbt build --select tag:order_metrics tag:hourly
```

## 使用上の注意

### タグは追加可能です

タグは階層的に蓄積されます。[前の例](/reference/resource-configs/tags#タグを使用してプロジェクトの各部分を実行する)では、次のようになります。

| Model                            | Tags                                  |
| -------------------------------- | ------------------------------------- |
| models/staging/stg_customers.sql | `contains_pii`, `hourly`              |
| models/staging/stg_payments.sql  | `contains_pii`, `hourly`, `finance`   |
| models/marts/dim_customers.sql   | `contains_pii`, `hourly`, `published` |
| models/metrics/daily_metrics.sql | `contains_pii`, `daily`, `published`  |

### その他のリソースタイプ

タグは、[ソース](/docs/build/sources)、[エクスポージャー](/docs/build/exposures)、さらにはリソース内の_特定の列_にも適用できます。
これらのリソースはまだ `config` プロパティをサポートしていないため、代わりに最上位キーとしてタグを指定する必要があります。

<File name='models/schema.yml'>

```yml
version: 2

exposures:
  - name: my_exposure
    tags: ['exposure_tag']
    ...

sources:
  - name: source_name
    tags: ['top_level']

    tables:
      - name: table_name
        tags: ['table_level']

        columns:
          - name: column_name
            tags: ['column_level']
            tests:
              - unique:
                  tags: ['test_level']
```

</File>


上記の例では、`unique` テストは次の 4 つのタグのいずれかによって選択されます:

```bash
dbt test --select tag:top_level
dbt test --select tag:table_level
dbt test --select tag:column_level
dbt test --select tag:test_level
```
