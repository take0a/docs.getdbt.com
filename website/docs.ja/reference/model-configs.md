---
title: Model 構成
description: "dbt で使用可能な model 構成のリファレンス ガイド。"
meta:
  resource_type: Models
---

import ConfigResource from '/snippets.ja/_config-description-resource.md';
import ConfigGeneral from '/snippets.ja/_config-description-general.md';

## 関連ドキュメント
* [モデル](/docs/build/models)
* [`run` コマンド](/reference/commands/run)

## 利用可能な構成

### モデル固有の構成

<ConfigResource meta={frontMatter.meta}/>

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Property file', value: 'property-yaml', },
    { label: 'Config block', value: 'config', },
  ]
}>
<TabItem value="project-yaml">

<File name='dbt_project.yml'>

<VersionBlock lastVersion="1.8">

```yaml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[materialized](/reference/resource-configs/materialized): <materialization_name>
    [+](/reference/resource-configs/plus-prefix)[sql_header](/reference/resource-configs/sql_header): <string>
    [+](/reference/resource-configs/plus-prefix)[on_configuration_change](/reference/resource-configs/on_configuration_change): apply | continue | fail # Only for materialized views on supported adapters
    [+](/reference/resource-configs/plus-prefix)[unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>
```

</VersionBlock>

<VersionBlock firstVersion="1.9">

```yaml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[materialized](/reference/resource-configs/materialized): <materialization_name>
    [+](/reference/resource-configs/plus-prefix)[sql_header](/reference/resource-configs/sql_header): <string>
    [+](/reference/resource-configs/plus-prefix)[on_configuration_change](/reference/resource-configs/on_configuration_change): apply | continue | fail # Only for materialized views on supported adapters
    [+](/reference/resource-configs/plus-prefix)[unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>
    [+](/reference/resource-configs/plus-prefix)[batch_size](/reference/resource-configs/batch-size): day | hour | month | year
    [+](/reference/resource-configs/plus-prefix)[begin](/reference/resource-configs/begin): "<ISO formatted date or datetime (like, "2024-01-15T12:00:00Z")>"
    [+](/reference/resource-configs/plus-prefix)[lookback](/reference/resource-configs/lookback): <integer>
    [+](/reference/resource-configs/plus-prefix)[concurrent_batches](/reference/resource-properties/concurrent_batches): true | false
```

</VersionBlock>
</File>

</TabItem>


<TabItem value="property-yaml">

<File name='models/properties.yml'>

<VersionBlock lastVersion="1.8">

```yaml
version: 2

models:
  - name: [<model-name>] #  Must match the filename of a model -- including case sensitivity.
    config:
      [materialized](/reference/resource-configs/materialized): <materialization_name>
      [sql_header](/reference/resource-configs/sql_header): <string>
      [on_configuration_change](/reference/resource-configs/on_configuration_change): apply | continue | fail # Only for materialized views on supported adapters
      [unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>

```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```yaml
version: 2

models:
  - name: [<model-name>] #  Must match the filename of a model -- including case sensitivity.
    config:
      [materialized](/reference/resource-configs/materialized): <materialization_name>
      [sql_header](/reference/resource-configs/sql_header): <string>
      [on_configuration_change](/reference/resource-configs/on_configuration_change): apply | continue | fail # Only for materialized views on supported adapters
      [unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>
      [batch_size](/reference/resource-configs/batch-size): day | hour | month | year
      [begin](/reference/resource-configs/begin): "<ISO formatted date or datetime (like, "2024-01-15T12:00:00Z")>"
      [lookback](/reference/resource-configs/lookback): <integer>
      [concurrent_batches](/reference/resource-properties/concurrent_batches): true | false

```
</VersionBlock>
</File>

</TabItem>

<TabItem value="config">

<File name='models/<model_name>.sql'>

<VersionBlock lastVersion="1.8">

```sql

{{ config(
    [materialized](/reference/resource-configs/materialized)="<materialization_name>",
    [sql_header](/reference/resource-configs/sql_header)="<string>"
    [on_configuration_change](/reference/resource-configs/on_configuration_change): apply | continue | fail #only for materialized views for supported adapters
    [unique_key](/reference/resource-configs/unique_key)='column_name_or_expression'
) }}

```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```sql

{{ config(
    [materialized](/reference/resource-configs/materialized)="<materialization_name>",
    [sql_header](/reference/resource-configs/sql_header)="<string>"
    [on_configuration_change](/reference/resource-configs/on_configuration_change): apply | continue | fail # Only for materialized views for supported adapters
    [unique_key](/reference/resource-configs/unique_key)='column_name_or_expression'
    [batch_size](/reference/resource-configs/batch-size)='day' | 'hour' | 'month' | 'year'
    [begin](/reference/resource-configs/begin)="<ISO formatted date or datetime (like, "2024-01-15T12:00:00Z")>"
    [lookback](/reference/resource-configs/lookback)= <integer>
    [concurrent_batches](/reference/resource-properties/concurrent_batches)= true | false
) }}

```

</VersionBlock>

</File>

</TabItem>

</Tabs>


### 一般的な構成

<ConfigGeneral />

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Property file', value: 'property-yaml', },
    { label: 'Config block', value: 'config', },
  ]
}>

<TabItem value="project-yaml">

<File name='dbt_project.yml'>

<VersionBlock lastVersion="1.8">

```yaml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[pre-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[post-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[database](/reference/resource-configs/database): <string>
    [+](/reference/resource-configs/plus-prefix)[schema](/reference/resource-properties/schema): <string>
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[persist_docs](/reference/resource-configs/persist_docs): <dict>
    [+](/reference/resource-configs/plus-prefix)[full_refresh](/reference/resource-configs/full_refresh): <boolean>
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[grants](/reference/resource-configs/grants): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[contract](/reference/resource-configs/contract): {<dictionary>}

```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```yaml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[pre-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[post-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[database](/reference/resource-configs/database): <string>
    [+](/reference/resource-configs/plus-prefix)[schema](/reference/resource-properties/schema): <string>
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[persist_docs](/reference/resource-configs/persist_docs): <dict>
    [+](/reference/resource-configs/plus-prefix)[full_refresh](/reference/resource-configs/full_refresh): <boolean>
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[grants](/reference/resource-configs/grants): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[contract](/reference/resource-configs/contract): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[event_time](/reference/resource-configs/event-time): my_time_field

```
</VersionBlock>
</File>

</TabItem>


<TabItem value="property-yaml">

<File name='models/properties.yml'>

<VersionBlock lastVersion="1.8">

```yaml
version: 2

models:
  - name: [<model-name>] # Must match the filename of a model -- including case sensitivity.
    config:
      [enabled](/reference/resource-configs/enabled): true | false
      [tags](/reference/resource-configs/tags): <string> | [<string>]
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [database](/reference/resource-configs/database): <string>
      [schema](/reference/resource-properties/schema): <string>
      [alias](/reference/resource-configs/alias): <string>
      [persist_docs](/reference/resource-configs/persist_docs): <dict>
      [full_refresh](/reference/resource-configs/full_refresh): <boolean>
      [meta](/reference/resource-configs/meta): {<dictionary>}
      [grants](/reference/resource-configs/grants): {<dictionary>}
      [contract](/reference/resource-configs/contract): {<dictionary>}
```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```yaml
version: 2

models:
  - name: [<model-name>] #  Must match the filename of a model -- including case sensitivity.
    config:
      [enabled](/reference/resource-configs/enabled): true | false
      [tags](/reference/resource-configs/tags): <string> | [<string>]
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [database](/reference/resource-configs/database): <string>
      [schema](/reference/resource-properties/schema): <string>
      [alias](/reference/resource-configs/alias): <string>
      [persist_docs](/reference/resource-configs/persist_docs): <dict>
      [full_refresh](/reference/resource-configs/full_refresh): <boolean>
      [meta](/reference/resource-configs/meta): {<dictionary>}
      [grants](/reference/resource-configs/grants): {<dictionary>}
      [contract](/reference/resource-configs/contract): {<dictionary>}
      [event_time](/reference/resource-configs/event-time): my_time_field
```

</VersionBlock>
</File>

</TabItem>

<TabItem value="config">

<File name='models/<model_name>.sql'>

<VersionBlock lastVersion="1.8">

```sql

{{ config(
    [enabled](/reference/resource-configs/enabled)=true | false,
    [tags](/reference/resource-configs/tags)="<string>" | ["<string>"],
    [pre_hook](/reference/resource-configs/pre-hook-post-hook)="<sql-statement>" | ["<sql-statement>"],
    [post_hook](/reference/resource-configs/pre-hook-post-hook)="<sql-statement>" | ["<sql-statement>"],
    [database](/reference/resource-configs/database)="<string>",
    [schema](/reference/resource-properties/schema)="<string>",
    [alias](/reference/resource-configs/alias)="<string>",
    [persist_docs](/reference/resource-configs/persist_docs)={<dict>},
    [meta](/reference/resource-configs/meta)={<dict>},
    [grants](/reference/resource-configs/grants)={<dict>},
    [contract](/reference/resource-configs/contract)={<dictionary>}
) }}

```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```sql

{{ config(
    [enabled](/reference/resource-configs/enabled)=true | false,
    [tags](/reference/resource-configs/tags)="<string>" | ["<string>"],
    [pre_hook](/reference/resource-configs/pre-hook-post-hook)="<sql-statement>" | ["<sql-statement>"],
    [post_hook](/reference/resource-configs/pre-hook-post-hook)="<sql-statement>" | ["<sql-statement>"],
    [database](/reference/resource-configs/database)="<string>",
    [schema](/reference/resource-properties/schema)="<string>",
    [alias](/reference/resource-configs/alias)="<string>",
    [persist_docs](/reference/resource-configs/persist_docs)={<dict>},
    [meta](/reference/resource-configs/meta)={<dict>},
    [grants](/reference/resource-configs/grants)={<dict>},
    [contract](/reference/resource-configs/contract)={<dictionary>},
    [event_time](/reference/resource-configs/event-time)='my_time_field',

) }}

```
</VersionBlock>

</File>

</TabItem>

</Tabs>

### ウェアハウス固有の構成

* [BigQuery 構成](/reference/resource-configs/bigquery-configs)
* [Redshift 構成](/reference/resource-configs/redshift-configs)
* [Snowflake 構成](/reference/resource-configs/snowflake-configs)
* [Databricks 構成](/reference/resource-configs/databricks-configs)
* [Spark 構成](/reference/resource-configs/spark-configs)

## モデルの構成

モデル構成は階層的に適用されます。インストール済みパッケージ内、および dbt プロジェクト内から、以下の方法でモデルを構成できます（優先順位順）。

1. モデル内で `config()` Jinja マクロを使用する。
2. `.yml` ファイルで `config` [リソースプロパティ](/reference/model-properties) を使用する。
3. `dbt_project.yml` プロジェクト ファイルの `models:` キーの下から。この場合、最も深くネストされたモデルが最も優先されます。
- モデル名の構成は、大文字と小文字の区別を含め、モデルの _filename_ と一致している必要があることに注意してください。大文字と小文字が一致していないと、dbt が構成を正しく適用できず、[dbt Explorer](/docs/collaborate/explore-projects) のメタデータに影響する可能性があります。

最も具体的な構成が常に優先されます。例えば、プロジェクトファイルでは、`marketing` サブディレクトリに適用された構成は、`jaffle_shop` プロジェクト全体に適用された構成よりも優先されます。モデルまたはモデルのディレクトリに構成を適用するには、[リソースパス](/reference/resource-configs/resource-path) をネストされた辞書キーとして定義します。

ルート dbt プロジェクト内のモデル構成は、インストール済みパッケージ内の構成よりも _高い_ 優先順位_ を持ちます。これにより、インストール済みパッケージの構成をオーバーライドして、dbt 実行をより細かく制御できるようになります。

## 例

### `dbt_project.yml` でのモデルディレクトリの設定

`dbt_project.yml` ファイルでモデルを設定するには、`models:` 設定オプションを使用します。設定にはプロジェクトの名前空間を指定してください（以下を参照）。

<File name='dbt_project.yml'>

```yml


name: dbt_labs

models:
  # Be sure to namespace your model configs to your project name
  dbt_labs:

    # This configures models found in models/events/
    events:
      +enabled: true
      +materialized: view

      # This configures models found in models/events/base
      # These models will be ephemeral, as the config above is overridden
      base:
        +materialized: ephemeral

      ...


```

</File>

### 設定を1つのモデルのみに適用する

一部の設定は特定のモデルに固有のものです。このような場合、`dbt_project.yml` ファイルに設定を配置するのは扱いにくい場合があります。代わりに、これらの設定をモデルの `.sql` ファイルの先頭、または個々の YAML プロパティに指定できます。

<File name='models/events/base/base_events.sql'>

```sql
{{
  config(
    materialized = "table",
    sort = 'event_time',
    dist = 'event_id'
  )
}}


select * from ...
```

</File>

<File name='models/events/base/properties.yml'>

```yaml
version: 2

models:
  - name: base_events
    config:
      materialized: table
      sort: event_time
      dist: event_id
```

</File>
