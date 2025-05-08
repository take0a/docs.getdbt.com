---
description: "Snapshot-name - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
---

<VersionBlock firstVersion="1.9">
<File name='snapshots/<filename>.yml'>

```yaml
snapshots:
  - name: snapshot_name
    relation: source('my_source', 'my_table')
    config:
      schema: string
      database: string
      unique_key: column_name_or_expression
      strategy: timestamp | check
      updated_at: column_name  # Required if strategy is 'timestamp'

```

</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

<File name='snapshots/<filename>.sql'>

```jinja2
{% snapshot snapshot_name %}

{% endsnapshot %}

```

</File>

import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>

</VersionBlock>

## 説明

[`ref` 関数](/reference/dbt-jinja-functions/ref) を使用して snapshot から選択する際に使用される snapshot の名前です。

この名前は、このプロジェクトまたはパッケージで定義されている他の「参照可能な」リソース（モデル、シード、他の snapshot ）の名前と競合してはなりません。

名前はファイル名と一致する必要はありません。したがって、 snapshot のファイル名は一意である必要はありません。

## 例
###  snapshot に「order_snapshot」という名前を付けます

<VersionBlock firstVersion="1.9">
<File name='snapshots/order_snapshot.yml'>


```yaml
snapshots:
  - name: order_snapshot
    relation: source('my_source', 'my_table')
    config:
      schema: string
      database: string
      unique_key: column_name_or_expression
      strategy: timestamp | check
      updated_at: column_name  # Required if strategy is 'timestamp'
```
</File>

</VersionBlock>

<VersionBlock lastVersion="1.8">
<File name='snapshots/orders.sql'>

```jinja2
{% snapshot orders_snapshot %}
...
{% endsnapshot %}

```

</File>

</VersionBlock>

下流モデルでこの snapshot から選択するには:

```sql
select * from {{ ref('orders_snapshot') }}
```
