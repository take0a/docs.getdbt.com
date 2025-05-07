---
id: "persist_docs"
description: "Persist_docs - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
datatype: Dict[Str, Bool]
---


<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Sources', value:'sources', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
  ]
}>

<TabItem value="models">

<File name='dbt_project.yml'>

```yml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +persist_docs:
      relation: true
      columns: true

```

</File>

<File name='models/<modelname>.sql'>

```sql

{{ config(
  persist_docs={"relation": true, "columns": true}
) }}

select ...

```

</File>

</TabItem>

<TabItem value="sources">

この設定は sources には実装されていません。

</TabItem>

<TabItem value="seeds">

<File name='dbt_project.yml'>

```yml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    +persist_docs:
      relation: true
      columns: true

```

</File>

</TabItem>

<TabItem value="snapshots">

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +persist_docs:
      relation: true
      columns: true

```

</File>

<VersionBlock firstVersion="1.9">
<File name='snapshots/snapshot_name.yml'>

```yaml
version: 2

snapshots:
  - name: snapshot_name
    [config](/reference/resource-properties/config):
      persist_docs:
        relation: true
        columns: true
```

</File>
</VersionBlock>

<File name='snapshots/<filename>.sql'>

```sql
{% snapshot [snapshot_name](snapshot_name) %}

{{ config(
  persist_docs={"relation": true, "columns": true}
) }}

select ...

{% endsnapshot %}

```

</File>

</TabItem>

</Tabs>

## 定義

オプションで、[リソースの説明](/reference/resource-properties/description)を列およびリレーションコメントとしてデータベースに保存します。デフォルトではドキュメントの保存は無効になっていますが、必要に応じて特定のリソースまたはリソースグループに対して有効にすることができます。

## サポート

`persist_docs` 構成は、最も広く使用されている dbt アダプタでサポートされています。
- Postgres
- Redshift
- Snowflake
- BigQuery
- Databricks
- Apache Spark

ただし、一部のデータベースでは、データベースオブジェクトに説明を追加できる場所と方法が制限されています。これらのデータベースアダプタは、`persist_docs` をサポートしていないか、部分的にしかサポートしていない可能性があります。

既知の問題と制限事項：

<WHCode>

<div warehouse="Databricks">

- 列レベルのコメントには `file_format: delta` (または別の「v2 ファイル形式」) が必要です

</div>

<div warehouse="Snowflake">

- 既知の問題はありません

</div>

</WHCode>

## Usage

### 列とリレーションのドキュメント化

モデルの[説明](/reference/resource-properties/description)を指定します:

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: dim_customers
    description: One record per customer
    columns:
      - name: customer_id
        description: Primary key

```

</File>

プロジェクト内の列とリレーションに対して `persist_docs` を有効にします:

<File name='dbt_project.yml'>

```yml
models:
  +persist_docs:
    relation: true
    columns: true
```

</File>

dbt を実行し、作成されたリレーションと列に説明が注釈付けされていることを確認します:

<Lightbox src="/img/reference/persist_docs_relation.png"
          title="Relation descriptions in BigQuery"/>

<Lightbox src="/img/reference/persist_docs_columns.png"
          title="Column descriptions in BigQuery"/>
