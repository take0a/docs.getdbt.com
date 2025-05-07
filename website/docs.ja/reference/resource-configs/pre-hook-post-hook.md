---
title: pre-hook と post-hook
description: "dbt でモデルが実行される前 (pre) と後 (post) に SQL を実行するためのフックを構成します。"
resource_types: [models, seeds, snapshots]
datatype: sql-statement | [sql-statement]
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

<Snippet path="post-and-pre-hooks-sql-statement" /> 

<File name='dbt_project.yml'>

```yml

models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +pre-hook: SQL-statement | [SQL-statement]
    +post-hook: SQL-statement | [SQL-statement]

```

</File>

<File name='models/<model_name>.sql'>

```sql

{{ config(
    pre_hook="SQL-statement" | ["SQL-statement"],
    post_hook="SQL-statement" | ["SQL-statement"],
) }}

select ...

```


</File>

<File name='models/properties.yml'>

```yml
models:
  - name: [<model_name>]
    config:
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
```

</File>

</TabItem>

<TabItem value="seeds">

<Snippet path="post-and-pre-hooks-sql-statement" /> 

<File name='dbt_project.yml'>

```yml

seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    +pre-hook: SQL-statement | [SQL-statement]
    +post-hook: SQL-statement | [SQL-statement]

```

</File>

<File name='seeds/properties.yml'>

```yml
seeds:
  - name: [<seed_name>]
    config:
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
```

</File>

</TabItem>

<TabItem value="snapshots">

<Snippet path="post-and-pre-hooks-sql-statement" /> 

<File name='dbt_project.yml'>

```yml

snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +pre-hook: SQL-statement | [SQL-statement]
    +post-hook: SQL-statement | [SQL-statement]

```

</File>

<VersionBlock lastVersion="1.8">

<File name='snapshots/<filename>.sql'>

```sql
{% snapshot snapshot_name %}
{{ config(
    pre_hook="SQL-statement" | ["SQL-statement"],
    post_hook="SQL-statement" | ["SQL-statement"],
) }}

select ...

{% end_snapshot %}

```

</File>
</VersionBlock>

<File name='snapshots/snapshot.yml'>

```yml
snapshots:
  - name: [<snapshot_name>]
    [config](/reference/resource-properties/config):
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
```

</File>

</TabItem>

</Tabs>

## 定義

モデル、シード、またはスナップショットの構築前または構築後に実行されるSQL文（またはSQL文のリスト）。

事前フックと事後フックは、SQL文を返すマクロを呼び出すこともできます。マクロが実行時にのみ利用可能な値（モデル設定や他のリソースへの `ref()` 呼び出しを入力として使用するなど）に依存する場合は、[マクロ呼び出しを中括弧で囲む](/ベストプラクティス/中括弧をネストしない#例外) 必要があります。

### フックを使用する理由

dbt は、必要なすべての定型 SQL（DDL、DML、DCL）を、すぐに使用できる機能を通じて提供することを目指しています。これらの機能は、迅速かつ簡潔に構成できます。場合によっては、データプラットフォームの機能に固有の SQL を実行したい、または実行する必要があるものの、dbt が（まだ）組み込み機能として提供していないことがあります。そのような場合、dbt のコンパイルコンテキストを使用して必要な SQL を正確に記述し、それをモデル、シード、またはスナップショットの前後に実行する `pre-` フックまたは `post-` フックに渡すことができます。

import SQLCompilationError from '/snippets.ja/_render-method.md';

<SQLCompilationError />

## Examples

### [Redshift] Unload one model to S3

<File name='model.sql'>

```sql
{{ config(
  post_hook = "unload ('select from {{ this }}') to 's3:/bucket_name/{{ this }}"
) }}

select ...
```

</File>

参照: [Redshift の `UNLOAD` に関するドキュメント](https://docs.aws.amazon.com/redshift/latest/dg/r_UNLOAD.html)

### [Apache Spark] Analyze tables after creation

<File name='dbt_project.yml'>

```yml

models:
  jaffle_shop: # this is the project name
    marts:
      finance:
        +post-hook:
          # this can be a list
          - "analyze table {{ this }} compute statistics for all columns"
          # or call a macro instead
          - "{{ analyze_table() }}"
```

参照: [Apache Spark の `ANALYZE TABLE` に関するドキュメント](https://spark.apache.org/docs/latest/sql-ref-syntax-aux-analyze-table.html)

</File>

### 追加の例

より詳細な例を[こちら](/docs/build/hooks-operations#additional-examples)にまとめました。

## 使用上の注意

### フックは累積的に適用されます

`dbt_project.yml` とモデルの `config` ブロックの両方でフックを定義した場合、両方のフックセットがモデルに適用されます。

### 実行順序

フックのインスタンスが複数定義されている場合、dbt は以下の順序で各フックを実行します。
1. 依存パッケージのフックは、アクティブパッケージのフックよりも先に実行されます。
2. モデル自体で定義されたフックは、`dbt_project.yml` で定義されたフックよりも後に実行されます。
3. 特定のコンテキスト内のフックは、定義された順序で実行されます。


### トランザクションの動作

トランザクションを使用するアダプタ（Postgres または Redshift）を使用している場合、デフォルトではフックはモデルの作成と同じトランザクション内で実行されることに注意してください。

これらのフックをトランザクションの _外部_ で実行する必要がある場合があります。たとえば、次のようになります。
* `post-hook` で `VACUUM` を実行したいが、トランザクション内では実行できない ([Redshift ドキュメント](https://docs.aws.amazon.com/redshift/latest/dg/r_VACUUM_command.html#r_VACUUM_usage_notes))
* 実行開始時に監査 <Term id="table" /> にレコードを挿入し、モデルの作成に失敗してもそのステートメントをロールバックしたくない場合。

この動作を実現するには、次のいずれかの構文を使用できます。
- 重要な注意: dbt がトランザクションをサポートしていないデータベースを使用している場合は、この構文を使用しないでください。これには、Snowflake、BigQuery、Spark、Databricks などのデータベースが含まれます。

<Tabs>
<TabItem value="beforebegin" label="Use before_begin and after_commit">

#### 設定ブロック: `before_begin` および `after_commit` ヘルパーマクロを使用する

<File name='models/<modelname>.sql'>

```sql
{{
  config(
    pre_hook=before_begin("SQL-statement"),
    post_hook=after_commit("SQL-statement")
  )
}}

select ...

```

</File>
</TabItem>

<TabItem value="dictionary" label="Use a dictionary">

#### 設定ブロック: 辞書​​を使用する

<File name='models/<modelname>.sql'>

```sql
{{
  config(
    pre_hook={
      "sql": "SQL-statement",
      "transaction": False
    },
    post_hook={
      "sql": "SQL-statement",
      "transaction": False
    }
  )
}}

select ...

```

</File>

</TabItem>

<TabItem value="dbt_project.yml" label="Use dbt_project.yml">

#### `dbt_project.yml`: 辞書を使用する

<File name='dbt_project.yml'>

```yml

models:
  +pre-hook:
    sql: "SQL-statement"
    transaction: false
  +post-hook:
    sql: "SQL-statement"
    transaction: false


```

</File>
</TabItem>
</Tabs>
