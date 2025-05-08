---
resource_types: [snapshots]
description: "Strategy - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
datatype: timestamp | check
---

<VersionBlock lastVersion="1.8">

import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>
</VersionBlock>

<Tabs
  defaultValue="timestamp"
  values={[
    { label: 'timestamp', value: 'timestamp', },
    { label: 'check', value: 'check', }
  ]
}>
<TabItem value="timestamp">

<VersionBlock firstVersion="1.9">

<File name='snapshots/<filename>.yml'>
  
  ```yaml
  snapshots:
  - [name: snapshot_name](/reference/resource-configs/snapshot_name):
    relation: source('my_source', 'my_table')
    config:
      strategy: timestamp
      updated_at: column_name
  ```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

<File name='snapshots/<filename>.sql'>

```jinja2
{% snapshot [snapshot_name](snapshot_name) %}

{{ config(
  strategy="timestamp",
  updated_at="column_name"
) }}

select ...

{% endsnapshot %}

```

</File>
</VersionBlock>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +strategy: timestamp
    +updated_at: column_name

```

</File>

</TabItem>

<TabItem value="check">

<VersionBlock firstVersion="1.9">

<File name='snapshots/<filename>.yml'>
  
  ```yaml
  snapshots:
  - [name: snapshot_name](/reference/resource-configs/snapshot_name):
    relation: source('my_source', 'my_table')
    config:
      strategy: check
      check_cols: [column_name] | "all"
  ```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">
<File name='snapshots/<filename>.sql'>

```jinja2
{% snapshot [snapshot_name](snapshot_name) %}

{{ config(
  strategy="check",
  check_cols=[column_name] | "all"
) }}

{% endsnapshot %}

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

</TabItem>

</Tabs>

## 説明

dbt がレコードの変更を検出するために使用する snapshot 戦略。2 つの違いを理解するには、[ snapshot ](/docs/build/snapshots#detecting-row-changes) のガイドをお読みください。

## デフォルト

これは**必須設定**です。デフォルト値はありません。

## 例

### timestamp strategy を使用する

<VersionBlock firstVersion="1.9">
<File name='snapshots/timestamp_example.yml'>

```yaml
snapshots:
  - name: orders_snapshot_timestamp
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      strategy: timestamp
      unique_key: id
      updated_at: updated_at

```

</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">
<File name='snapshots/timestamp_example.sql'>

```sql
{% snapshot orders_snapshot_timestamp %}

    {{
        config(
          target_schema='snapshots',
          strategy='timestamp',
          unique_key='id',
          updated_at='updated_at',
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>
</VersionBlock>


### check strategy を使用する

<VersionBlock firstVersion="1.9">
<File name='snapshots/check_example.yml'>

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
</VersionBlock>

<VersionBlock lastVersion="1.8">

```sql
{% snapshot orders_snapshot_check %}

    {{
        config(
          target_schema='snapshots',
          strategy='check',
          unique_key='id',
          check_cols=['status', 'is_cancelled'],
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```
</VersionBlock>

### 上級編：カスタムスナップショット戦略の定義と使用

 snapshot 戦略は、`snapshot_<strategy>_strategy` という名前のマクロとして実装されています。
* タイムスタンプ戦略の [ソースコード](https://github.com/dbt-labs/dbt-adapters/blob/60005a0a2bd33b61cb65a591bc1604b1b3fd25d5/dbt/include/global_project/macros/materializations/snapshots/strategies.sql#L52)
* [ソースチェック戦略のコード](https://github.com/dbt-labs/dbt-adapters/blob/60005a0a2bd33b61cb65a591bc1604b1b3fd25d5/dbt/include/global_project/macros/materializations/snapshots/strategies.sql#L136)

同じ命名パターンを持つマクロをプロジェクトに追加することで、独自の snapshot 戦略を実装できます。例えば、物理削除を記録する戦略「timestamp_with_deletes」を作成できます。

1. 「snapshot_timestamp_with_deletes_strategy」というマクロを作成します。既存のコードを参考に、必要に応じて調整してください。
2. この戦略を「strategy」設定から使用します。

<VersionBlock firstVersion="1.9">
<File name='snapshots/<filename>.yml'>

```yaml
snapshots:
  - name: my_custom_snapshot
    relation: source('my_source', 'my_table')
    config:
      strategy: timestamp_with_deletes
      updated_at: updated_at_column
      unique_key: id
```
</File>
</VersionBlock>


<VersionBlock lastVersion="1.8">
<File name='snapshots/<filename>.sql'>

```jinja2
{% snapshot [snapshot_name](snapshot_name) %}

{{ config(
  strategy="timestamp_with_deletes",
  updated_at="column_name"
) }}

{% endsnapshot %}

```

</File>
</VersionBlock>
