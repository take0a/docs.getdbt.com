---
resource_types: [snapshots, models]
description: "dbt の unique_key 構成について詳しく学びます。"
datatype: column_name_or_expression
intro_text: "unique_key は増分モデルまたはスナップショットのレコードを識別し、変更が正しくキャプチャまたは更新されることを保証します。"
---


<Tabs>

<TabItem value="models" label="Models">

[増分モデルの](/docs/build/incremental-models) SQL ファイルの `config` ブロック、`models/properties.yml` ファイル、または `dbt_project.yml` ファイルで `unique_key` を構成します。

<File name='models/my_incremental_model.sql'>

```sql
{{
    config(
        materialized='incremental',
        unique_key='id'
    )
}}

```

</File>

<File name='models/properties.yml'>

```yaml
models:
  - name: my_incremental_model
    description: "An incremental model example with a unique key."
    config:
      materialized: incremental
      unique_key: id

```

</File>

<File name='dbt_project.yml'>

```yaml
name: jaffle_shop

models:
  jaffle_shop:
    staging:
      +unique_key: id
```

</File>

</TabItem>

<TabItem value="snapshots" label="Snapshots">

<VersionBlock firstVersion="1.9">

[スナップショット](/docs/build/snapshots)の場合は、`snapshot/filename.yml` ファイルまたは `dbt_project.yml` ファイルで `unique_key` を設定します。

<File name='snapshots/<filename>.yml'>

```yaml
snapshots:
  - name: orders_snapshot
    relation: source('my_source', 'my_table')
    [config](/reference/snapshot-configs):
      unique_key: order_id

```

</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

Configure the `unique_key` in the `config` block of your snapshot SQL file or in your `dbt_project.yml` file.

import SnapshotYaml from '/snippets/_snapshot-yaml-spec.md';

<SnapshotYaml/>

<File name='snapshots/<filename>.sql'>

```jinja2
{{ config(
  unique_key="column_name"
) }}

```
</File>
</VersionBlock>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +unique_key: column_name_or_expression

```

</File>

</TabItem>
</Tabs>

## 説明

スナップショットまたは増分モデルの入力に含まれる各レコードを一意に識別する列名または式。dbt はこのキーを使用して、入力レコードをターゲットテーブル（スナップショットまたは増分モデル）内の既存のレコードと照合し、変更を正しくキャプチャまたは更新します。
* 増分モデルの場合、dbt は古い行を置き換えます（マージキーや upsert のように）。
* スナップショットの場合、dbt は履歴を保持し、時間の経過とともに変化する同じ `unique_key` の複数の行を保存します。

dbt Cloud の「最新」リリーストラックおよび dbt v1.9 以降では、[スナップショット](/docs/build/snapshots) は `snapshots/` ディレクトリ内の YAML ファイルで定義および構成されます。スナップショット YAML ファイルの `config` キー内で、1 つまたは複数の `unique_key` 値を指定できます。

:::caution 

一意でないキーを指定すると、予期しないスナップショット結果になります。dbt は**このキーの一意性をテストしません**。このキーが実際に一意であることを確認するために、ソース データの[テスト](/blog/primary-key-testing#how-to-test-primary-keys-with-dbt)を検討してください。
:::

## デフォルト

これは**必須パラメータ**です。デフォルトは指定されていません。


## 例
### `id` 列を一意のキーとして使用する

<Tabs>

<TabItem value="models" label="Models">

この例では、`id` 列は増分モデルの一意のキーです。

<File name='models/my_incremental_model.sql'>

```sql
{{
    config(
        materialized='incremental',
        unique_key='id'
    )
}}

select * from ..
```

</File>
</TabItem>

<TabItem value="snapshots" label="Snapshots">

この例では、`id` 列がスナップショットの一意のキーとして使用されます。

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
<File name='snapshots/<filename>.sql'>

```jinja2
{{
    config(
      unique_key="id"
    )
}}

```

</File>

これをYAMLで記述することもできます。複数のスナップショットが同じ `unique_key` を共有する場合は、これが良いかもしれません（ただし、上記のように、この設定は設定ブロックで適用することをお勧めします）。
</VersionBlock>

You can also specify configurations in your `dbt_project.yml` file if multiple snapshots share the same `unique_key`:

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +unique_key: id

```

</File>

</TabItem>
</Tabs>

<VersionBlock firstVersion="1.9">

### 複数の一意のキーを使用する

<Tabs>
<TabItem value="models" label="Models">

増分モデルに複数の一意のキーを設定する場合は、単一の列を表す文字列、または組み合わせて使用​​できる一重引用符で囲まれた列名のリスト（例：['col1', 'col2', …]）を指定します。

列にnull値を含めることはできません。null値を含めると、増分モデルは行のマッチングに失敗し、重複行を生成します。詳細については、[一意のキーの定義](/docs/build/incremental-models#defining-a-unique-key-optional)を参照してください。

<File name='models/my_incremental_model.sql'>

```sql
{{ config(
    materialized='incremental',
    unique_key=['order_id', 'location_id']
) }}

with...

```

</File>

</TabItem>

<TabItem value="snapshots" label="Snapshots">

`primary_key` 列に複数の一意のキーを使用するようにスナップショットを設定できます。

<File name='snapshots/transaction_items_snapshot.yml'>

```yaml
snapshots:
  - name: orders_snapshot
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: 
        - order_id
        - product_id
      strategy: timestamp
      updated_at: updated_at
      
```

</File>
</TabItem>
</Tabs>
</VersionBlock>

<VersionBlock lastVersion="1.8">

### Use a combination of two columns as a unique key

<Tabs>
<TabItem value="models" label="Models">

<File name='models/my_incremental_model.sql'>

```sql
{{ config(
    materialized='incremental',
    unique_key=['order_id', 'location_id']
) }}

with...

```

</File>

</TabItem>

<TabItem value="snapshots" label="Snapshots">

This configuration accepts a valid column expression. As such, you can concatenate two columns together as a unique key if required. It's a good idea to use a separator (for example, `'-'`) to ensure uniqueness.

<File name='snapshots/transaction_items_snapshot.sql'>

```jinja2
{% snapshot transaction_items_snapshot %}

    {{
        config(
          unique_key="transaction_id||'-'||line_item_id",
          ...
        )
    }}

select
    transaction_id||'-'||line_item_id as id,
    *
from {{ source('erp', 'transactions') }}

{% endsnapshot %}

```

</File>

Though, it's probably a better idea to construct this column in your query and use that as the `unique_key`:

<File name='models/transaction_items_ephemeral.sql'>

```sql
{{ config(materialized='ephemeral') }}

select
  transaction_id || '-' || line_item_id as id,
  *
from {{ source('erp', 'transactions') }}

```

</File>

In this example, we create an ephemeral model `transaction_items_ephemeral` that creates an `id` column that can be used as the `unique_key` our snapshot configuration.

<File name='snapshots/transaction_items_snapshot.sql'>

```jinja2

{% snapshot transaction_items_snapshot %}

    {{
        config(
          unique_key="id",
          ...
        )
    }}

select
    transaction_id || '-' || line_item_id as id,
    *
from {{ source('erp', 'transactions') }}

{% endsnapshot %}


```

</File>
</TabItem>
</Tabs>
</VersionBlock>
