---
resource_types: [snapshots]
description: "dbt の check_cols 構成を理解するには、このガイドをお読みください。"
datatype: "[column_name] | all"
---

<VersionBlock firstVersion="1.9">
<File name="snapshots/<filename>.yml">
  
  ```yml
  snapshots:
  - name: snapshot_name
    relation: source('my_source', 'my_table')
    config:
      schema: string
      unique_key: column_name_or_expression
      strategy: check
      check_cols:
        - column_name
  ```
  
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>

<File name='snapshots/<filename>.sql'>

```jinja2
{{ config(
  strategy="check",
  check_cols=["column_name"]
) }}

```

</File>
</VersionBlock>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +strategy: check
    +check_cols: [column_name] | all

```

</File>

## 説明

 snapshot クエリの結果内で変更を確認する列のリストです。

または、`all` 値を使用してすべての列を使用することもできます（ただし、パフォーマンスが低下する可能性があります）。

このパラメータは、**`check` [strategy](/reference/resource-configs/strategy)** を使用する場合に必須です。

## デフォルト

デフォルトは指定されていません。

## 例

### 列のリストの変更を確認する

<VersionBlock firstVersion="1.9">

<File name="snapshots/orders_snapshot_check.yml">

```yaml
snapshots:
  - name: orders_snapshot_check
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: id
      strategy: check
      check_cols:
        - status
        - is_cancelled
```
</File>

下流モデルでこの snapshot から選択するには: `select * from {{ ref('orders_snapshot_check') }}`
</VersionBlock>

<VersionBlock lastVersion="1.8">

```sql
{% snapshot orders_snapshot_check %}

    {{
        config(
          strategy='check',
          unique_key='id',
          check_cols=['status', 'is_cancelled'],
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</VersionBlock>

### すべての列の変更を確認する

<VersionBlock firstVersion="1.9">

<File name="orders_snapshot_check.yml">

```yaml
snapshots:
  - name: orders_snapshot_check
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: id
      strategy: check
      check_cols: all
  ```
</File>

下流モデルでこの snapshot から選択するには: `select * from {{{ ref('orders_snapshot_check') }}`
</VersionBlock>

<VersionBlock lastVersion="1.8">

```sql
{% snapshot orders_snapshot_check %}

    {{
        config(
          strategy='check',
          unique_key='id',
          check_cols='all',
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```
</VersionBlock>
