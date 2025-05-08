---
resource_types: [snapshots]
description: "Updated_at - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
datatype: column_name
---


<VersionBlock firstVersion="1.9">

<File name="snapshots/snapshots.yml">

```yaml
snapshots:
  - name: snapshot
    relation: source('my_source', 'my_table')
    [config](/reference/snapshot-configs):
      strategy: timestamp
      updated_at: column_name
```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>

<File name='snapshots/<filename>.sql'>

```jinja2
{{ config(
  strategy="timestamp",
  updated_at="column_name"
) }}

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

<VersionBlock firstVersion="1.9">

:::caution

`updated_at` 列のデータ型がアダプタで設定されたデフォルトと一致しない場合は警告が表示されます。

:::

</VersionBlock>

## 説明

スナップショットクエリの結果内の、レコード行が最後に更新された日時を表す列。

このパラメータは**`timestamp` [strategy](/reference/resource-configs/strategy)**を使用する場合に必須です。`updated_at`フィールドは、使用するデータプラットフォームに応じて、ISO日付文字列とUnixエポック整数をサポートする場合があります。


## デフォルト

デフォルトは指定されていません。

## 例

### 列名 `updated_at` を使用する

<VersionBlock firstVersion="1.9">

<File name="snapshots/orders_snapshot.yml">

```yaml
snapshots:
  - name: orders_snapshot
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: id
      strategy: timestamp
      updated_at: updated_at

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
      unique_key='id',

      strategy='timestamp',
      updated_at='updated_at'
    )
}}

select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}

```

</File>
</VersionBlock>

### 2つの列を結合して、信頼性の高い `updated_at` 列を作成します。

レコードが更新された場合にのみ `updated_at` 列が入力されるデータソースを考えてみましょう（つまり、「null」値は、レコードが作成後に更新されていないことを示します）。

`updated_at` 設定は式ではなく列名のみを受け入れるため、結合された列を含めるようにスナップショットクエリを更新する必要があります。


<VersionBlock firstVersion="1.9">

1. 変換を実行するためのステージングモデルを作成します。
  `models/` ディレクトリに、`updated_at` 列と `created_at` 列を新しい列 `updated_at_for_snapshot` に結合してステージングモデルを構成する SQL ファイルを作成します。

    <File name='models/staging_orders.sql'>

    ```sql
    select  * coalesce (updated_at, created_at) as updated_at_for_snapshot
    from {{ source('jaffle_shop', 'orders') }}

    ```
    </File>

2. YAMLファイルでスナップショット設定を定義します。
  `snapshots/`ディレクトリに、スナップショットを定義し、先ほど作成した`updated_at_for_snapshot`ステージングモデルを参照するYAMLファイルを作成します。

    <File name="snapshots/orders_snapshot.yml">

    ```yaml
    snapshots:
      - name: orders_snapshot
        relation: ref('staging_orders')
        config:
          schema: snapshots
          unique_key: id
          strategy: timestamp
          updated_at: updated_at_for_snapshot

    ```
    </File>

3. `dbt snapshot` を実行してスナップショットを実行します。

あるいは、必要な変換を実行するための一時的なモデルを作成することもできます。その場合、スナップショットの `relation` キーでこのモデルを参照します。

</VersionBlock>


<VersionBlock lastVersion="1.8">

<File name='snapshots/orders.sql'>

```sql
{% snapshot orders_snapshot %}

{{
    config(
      target_schema='snapshots',
      unique_key='id',

      strategy='timestamp',
      updated_at='updated_at_for_snapshot'
    )
}}

select
    *,
    coalesce(updated_at, created_at) as updated_at_for_snapshot

from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}

```

</File>
</VersionBlock>

