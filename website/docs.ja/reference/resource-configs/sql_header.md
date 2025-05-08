---
resource_types: [models]
description: "Sql_header - Read this in-depth guide to learn about configurations in dbt."
datatype: "string"
---

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
  ]
}>
<TabItem value="models">

<File name='models/<modelname>.sql'>

```sql
{{ config(
  sql_header="<sql-statement>"
) }}

select ...


```

</File>

<File name='dbt_project.yml'>

```yml
[config-version](/reference/project-configs/config-version): 2

models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +sql_header: <sql-statement>

```

</File>

</TabItem>


<TabItem value="seeds">

この設定は seeds には実装されていません

</TabItem>

<TabItem value="snapshots">

<File name='snapshots/<filename>.sql'>

```sql
{% snapshot [snapshot_name](snapshot_name) %}

{{ config(
  sql_header="<sql-statement>"
) }}

select ...

{% endsnapshot %}

```

</File>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +sql_header: <sql-statement>

```

</File>

</TabItem>

</Tabs>


## 定義

モデルとスナップショットの構築時に dbt が実行する `create table as` および `create view as` ステートメントの上に SQL を挿入するためのオプション設定です。

`sql_header` は、設定を使用するか、`set_sql_header` マクロを `call` することで設定できます（以下の例を参照）。

## 事前フックとの比較

[事前フック](/reference/resource-configs/pre-hook-post-hook) は、モデル作成前に S​​QL を _先行_ クエリとして実行する機会も提供します。これに対し、`sql_header` 内の SQL は、`create table|view as` ステートメントと同じ _クエリ_ で実行されます。

その結果、[Snowflake セッションパラメータ](https://docs.snowflake.com/en/sql-reference/parameters.html) や [BigQuery 一時 UDF](https://cloud.google.com/bigquery/docs/reference/standard-sql/user-defined-functions#sql-udf-examples) でより便利になります。

## 例

### 特定のモデルのSnowflakeセッションパラメータを設定します。

これにはconfigブロック構文を使用します。

<File name='models/my_model.sql'>

```sql
{{ config(
  sql_header="alter session set timezone = 'Australia/Sydney';"
) }}

select * from {{ ref('other_model') }}
```

</File>

### すべてのモデルのSnowflakeセッションパラメータを設定する

<File name='dbt_project.yml'>

```yml
config-version: 2

models:
  +sql_header: "alter session set timezone = 'Australia/Sydney';"
```

</File>

### BigQuery の一時 UDF を作成します。

この例では、`set_sql_header` マクロを呼び出します。このマクロは、複数行の SQL 文を挿入する必要がある場合に使用できる便利なラッパーです。この場合、`sql_header` 構成キーを使用する必要はありません。

<File name='models/my_model.sql'>

```sql
-- Supply a SQL header:
{% call set_sql_header(config) %}
  CREATE TEMPORARY FUNCTION yes_no_to_boolean(answer STRING)
  RETURNS BOOLEAN AS (
    CASE
    WHEN LOWER(answer) = 'yes' THEN True
    WHEN LOWER(answer) = 'no' THEN False
    ELSE NULL
    END
  );
{%- endcall %}

-- Supply your model code:


select yes_no_to_boolean(yes_no) from {{ ref('other_model') }}
```

</File>
