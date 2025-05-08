---
resource_types: [snapshots]
description: "Target_schema - Read this in-depth guide to learn about configurations in dbt."
datatype: string
---

:::note

dbt Core v1.9以降では、この機能は利用できなくなりました。`generate_schema_name`マクロを尊重しつつカスタムスキーマを定義するには、代わりに[schema](/reference/resource-configs/schema)設定を使用してください。

[dbt Cloud "最新" リリーストラック](/docs/dbt-versions/cloud-release-tracks)で今すぐお試しください。

:::

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +target_schema: string

```

</File>

<File name='snapshots/<filename>.sql'>

```jinja2
{{ config(
      target_schema="string"
) }}

```

</File>

## Description
The schema that dbt should build a [snapshot](/docs/build/snapshots) <Term id="table" /> into. When `target_schema` is provided, snapshots build into the same `target_schema`, no matter who is running them.

On **BigQuery**, this is analogous to a `dataset`.

## Default

<VersionBlock lastVersion="1.8" >This is a required parameter, no default is provided. </VersionBlock>
<VersionBlock firstVersion="1.9.1">In dbt Core v1.9+ and dbt Cloud "Latest" release track, this is not a required parameter. </VersionBlock>

## Examples
### Build all snapshots in a schema named `snapshots`

<File name='dbt_project.yml'>

```yml
snapshots:
  +target_schema: snapshots

```

</File>

<VersionBlock lastVersion="1.8" >

### Use the same schema-naming behavior as models

For native support of environment-aware snapshots, upgrade to dbt Core version 1.9+ and remove any existing `target_schema` configuration. 

</VersionBlock>
