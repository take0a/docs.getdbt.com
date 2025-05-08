---
title: invalidate_hard_deletes
resource_types: [snapshots]
description: "Invalidate_hard_deletes - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
datatype: column_name
sidebar_label: invalidate_hard_deletes
---

# invalidate_hard_deletes <Lifecycle status="legacy" />

<IntroText>

クエリの snapshot 中にハード削除されたレコードを無効にできるようにするレガシー オプトイン構成。

</IntroText>

:::warning これはレガシー構成です。代わりに [`hard_deletes`](/reference/resource-configs/hard-deletes) 構成を使用してください。

dbt Cloud リリース トラックおよび dbt Core 1.9 以降では、ソースから削除された行の処理方法をより適切に制御するために、`invalidate_hard_deletes` 構成が [`hard_deletes`](/reference/resource-configs/hard-deletes) 構成に置き換えられています。

新しい snapshot の場合は、構成を `invalidate_hard_deletes=true` ではなく `hard_deletes='invalidate'` に設定してください。既存の snapshot の場合は、この設定を有効にする前に、既存のテーブルの更新を行ってください。
:::

<VersionBlock firstVersion="1.9">

<File name='snapshots/<filename>.yml'>

```yaml
snapshots:
  - name: snapshot
    relation: source('my_source', 'my_table')
    [config](/reference/snapshot-configs):
      strategy: timestamp
      invalidate_hard_deletes: true | false
```

</File>


</VersionBlock>

<VersionBlock lastVersion="1.8">

import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>

<File name='snapshots/<filename>.sql'>

```jinja2
{{
  config(
    strategy="timestamp",
    invalidate_hard_deletes=True
  )
}}

```

</File>
</VersionBlock>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +strategy: timestamp
    +invalidate_hard_deletes: true

```

</File>

## 説明
クエリの snapshot 作成時に、物理的に削除されたレコードを無効化できるようにするオプトイン機能。


## デフォルト
デフォルトではこの機能は無効になっています。

## 例

<VersionBlock firstVersion="1.9">
<File name='snapshots/orders.yml'>

```yaml
snapshots:
  - name: orders_snapshot
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      database: analytics
      unique_key: id
      strategy: timestamp
      updated_at: updated_at
      invalidate_hard_deletes: true
  ```
</File>

</VersionBlock>

<VersionBlock lastVersion="1.8">
<File name='snapshots/orders.sql'>

```sql
{% snapshot orders_snapshot %}

    {{
        config(
          target_schema='snapshots',
          strategy='timestamp',
          unique_key='id',
          updated_at='updated_at',
          invalidate_hard_deletes=True,
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>
</VersionBlock>
