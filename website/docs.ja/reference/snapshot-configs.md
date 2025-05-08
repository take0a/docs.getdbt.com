---
title: Snapshot 構成
description: "dbt での snapshot 構成の使用については、このガイドをお読みください。"
meta:
  resource_type: Snapshots
---

import ConfigResource from '/snippets.ja/_config-description-resource.md';
import ConfigGeneral from '/snippets.ja/_config-description-general.md';

## 関連ドキュメント

* [ snapshot ](/docs/build/snapshots)
* `dbt snapshot` [コマンド](/reference/commands/snapshot)

## 利用可能な構成
### Snapshot-specific configurations

<ConfigResource meta={frontMatter.meta} />

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'YAML file', value: 'property-yaml', },
    { label: 'Config block', value: 'config-resource', },
  ]
}>

<TabItem value="project-yaml">

<VersionBlock lastVersion="1.8">

<File name='dbt_project.yml'>

```yaml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[target_schema](/reference/resource-configs/target_schema): <string>
    [+](/reference/resource-configs/plus-prefix)[target_database](/reference/resource-configs/target_database): <string>
    [+](/reference/resource-configs/plus-prefix)[unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>
    [+](/reference/resource-configs/plus-prefix)[strategy](/reference/resource-configs/strategy): timestamp | check
    [+](/reference/resource-configs/plus-prefix)[updated_at](/reference/resource-configs/updated_at): <column_name>
    [+](/reference/resource-configs/plus-prefix)[check_cols](/reference/resource-configs/check_cols): [<column_name>] | all
    [+](/reference/resource-configs/plus-prefix)[invalidate_hard_deletes](/reference/resource-configs/invalidate_hard_deletes) : true | false
```

</File>

</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='dbt_project.yml'>

```yaml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[schema](/reference/resource-configs/schema): <string>
    [+](/reference/resource-configs/plus-prefix)[database](/reference/resource-configs/database): <string>
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>
    [+](/reference/resource-configs/plus-prefix)[strategy](/reference/resource-configs/strategy): timestamp | check
    [+](/reference/resource-configs/plus-prefix)[updated_at](/reference/resource-configs/updated_at): <column_name>
    [+](/reference/resource-configs/plus-prefix)[check_cols](/reference/resource-configs/check_cols): [<column_name>] | all
    [+](/reference/resource-configs/plus-prefix)[snapshot_meta_column_names](/reference/resource-configs/snapshot_meta_column_names): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[dbt_valid_to_current](/reference/resource-configs/dbt_valid_to_current): <string> 
    [+](/reference/resource-configs/plus-prefix)[hard_deletes](/reference/resource-configs/hard-deletes): string
```

</File>

</VersionBlock>

</TabItem>

<TabItem value="property-yaml">

<VersionBlock lastVersion="1.8">

**Note:** Required snapshot properties _will not_ work when only defined in `config` YAML blocks. We recommend that you define these in `dbt_project.yml` or a `config()` block within the snapshot `.sql` file or upgrade to v1.9.

</VersionBlock>

<VersionBlock firstVersion="1.9">
  
利用可能な構成については、[ snapshot の構成](/docs/build/snapshots#configuring-snapshots)を参照してください。

<File name='snapshots/schema.yml'>

```yml
snapshots:
  - name: <string>
    config:
      [database](/reference/resource-configs/database): <string>
      [schema](/reference/resource-configs/schema): <string>
      [unique_key](/reference/resource-configs/unique_key): <column_name_or_expression>
      [strategy](/reference/resource-configs/strategy): timestamp | check
      [updated_at](/reference/resource-configs/updated_at): <column_name>
      [check_cols](/reference/resource-configs/check_cols): [<column_name>] | all
      [snapshot_meta_column_names](/reference/resource-configs/snapshot_meta_column_names): {<dictionary>}
      [hard_deletes](/reference/resource-configs/hard-deletes): string
      [dbt_valid_to_current](/reference/resource-configs/dbt_valid_to_current): <string>
```
</File>

</VersionBlock>

</TabItem>

<TabItem value="config-resource">

import LegacySnapshotConfig from '/snippets.ja/_legacy-snapshot-config.md';

<LegacySnapshotConfig />

<VersionBlock lastVersion="1.8">

```jinja

{{ config(
    [target_schema](/reference/resource-configs/target_schema)="<string>",
    [target_database](/reference/resource-configs/target_database)="<string>",
    [unique_key](/reference/resource-configs/unique_key)="<column_name_or_expression>",
    [strategy](/reference/resource-configs/strategy)="timestamp" | "check",
    [updated_at](/reference/resource-configs/updated_at)="<column_name>",
    [check_cols](/reference/resource-configs/check_cols)=["<column_name>"] | "all"
    [invalidate_hard_deletes](/reference/resource-configs/invalidate_hard_deletes) : true | false
) }}

```
</VersionBlock>

</TabItem>

</Tabs>

###  snapshot 設定の移行

dbt Core v1.9 で導入された最新の snapshot 設定（[`snapshot_meta_column_names`](/reference/resource-configs/snapshot_meta_column_names)、[`dbt_valid_to_current`](/reference/resource-configs/dbt_valid_to_current)、`hard_deletes` など）は、新しい snapshot に最適です。既存の snapshot については、 snapshot 間の不整合を回避するために、以下の設定を推奨します。

#### 既存の snapshot の場合

- テーブルの移行 - 以前の snapshot を新しいテーブルスキーマと値に移行します。
  -  snapshot のバックアップコピーを作成します。
  - 必要に応じて `alter` ステートメント（または `alter` ステートメントを適用するスクリプト）を使用して、テーブルの整合性を確保します。
- 新しい構成 - 構成を 1 つずつ変換し、テストしながら進めます。

:::warning
データを移行せずに `dbt_valid_to_current` などの最新の構成のいずれかを使用すると、古いデータと新しいデータが混在し、ダウンストリームの結果が不正確になる可能性があります。
:::

### General configurations

<ConfigGeneral />


<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'YAML file', value: 'property-yaml', },
    { label: 'Config block', value: 'config', },
  ]
}>
<TabItem value="project-yaml">

<File name='dbt_project.yml'>

<VersionBlock firstVersion="1.9">


```yaml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[pre-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[post-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[persist_docs](/reference/resource-configs/persist_docs): {<dict>}
    [+](/reference/resource-configs/plus-prefix)[grants](/reference/resource-configs/grants): {<dict>}
    [+](/reference/resource-configs/plus-prefix)[event_time](/reference/resource-configs/event-time): my_time_field
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[pre-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[post-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[persist_docs](/reference/resource-configs/persist_docs): {<dict>}
    [+](/reference/resource-configs/plus-prefix)[grants](/reference/resource-configs/grants): {<dict>}
```
</VersionBlock>
</File>

</TabItem>

<TabItem value="property-yaml">

<VersionBlock lastVersion="1.8">

<File name='snapshots/properties.yml'>

```yaml
version: 2

snapshots:
  - name: [<snapshot-name>]
    config:
      [enabled](/reference/resource-configs/enabled): true | false
      [tags](/reference/resource-configs/tags): <string> | [<string>]
      [alias](/reference/resource-configs/alias): <string>
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [persist_docs](/reference/resource-configs/persist_docs): {<dict>}
      [grants](/reference/resource-configs/grants): {<dictionary>}
```

</File>
</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='snapshots/properties.yml'>

```yaml
version: 2

snapshots:
  - name: [<snapshot-name>]
    relation: source('my_source', 'my_table')
    config:
      [enabled](/reference/resource-configs/enabled): true | false
      [tags](/reference/resource-configs/tags): <string> | [<string>]
      [alias](/reference/resource-configs/alias): <string>
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [persist_docs](/reference/resource-configs/persist_docs): {<dict>}
      [grants](/reference/resource-configs/grants): {<dictionary>}
      [event_time](/reference/resource-configs/event-time): my_time_field
```

</File>
</VersionBlock>

</TabItem>

<TabItem value="config">

<LegacySnapshotConfig />

<VersionBlock lastVersion="1.8">

```jinja

{{ config(
    [enabled](/reference/resource-configs/enabled)=true | false,
    [tags](/reference/resource-configs/tags)="<string>" | ["<string>"],
    [alias](/reference/resource-configs/alias)="<string>", 
    [pre_hook](/reference/resource-configs/pre-hook-post-hook)="<sql-statement>" | ["<sql-statement>"],
    [post_hook](/reference/resource-configs/pre-hook-post-hook)="<sql-statement>" | ["<sql-statement>"]
    [persist_docs](/reference/resource-configs/persist_docs)={<dict>}
    [grants](/reference/resource-configs/grants)={<dict>}
) }}

```

</VersionBlock>

</TabItem>

</Tabs>

##  snapshot の設定

 snapshot は複数の方法で設定できます。

<VersionBlock firstVersion="1.9">

1. YAMLファイル内の`config` [リソースプロパティ](/reference/model-properties)を使用して定義されます。通常は[ snapshot ディレクトリ](/reference/project-configs/snapshot-paths)または任意のフォルダに保存されます。[dbt Cloudリリーストラック](/docs/dbt-versions/cloud-release-tracks)、dbt v1.9以降で利用可能です。
2. `dbt_project.yml`ファイルの`snapshots:`キーの下にあります。 snapshot または snapshot のディレクトリに構成を適用するには、リソースパスをネストされた辞書キーとして定義します。
</VersionBlock>

<VersionBlock lastVersion="1.8">

1. Using a `config` block within a snapshot defined in Jinja SQL.
2. From the `dbt_project.yml` file, under the `snapshots:` key. To apply a configuration to a snapshot, or directory of snapshots, define the resource path as nested dictionary keys.
3. Defined in a YAML file using the `config` [resource property](/reference/model-properties), typically in your [snapshots directory](/reference/project-configs/snapshot-paths) (available in  [the dbt Cloud "Latest" release track](/docs/dbt-versions/cloud-release-tracks) and dbt v1.9 and higher).
</VersionBlock>

Snapshot configurations are applied hierarchically in the order above with higher taking precedence. You can also apply [tests](/reference/snapshot-properties) to snapshots using the [`tests` property](/reference/resource-properties/data-tests).

### 例

<VersionBlock firstVersion="1.9">
次の例は、`dbt_project.yml` ファイルと `.yml` ファイルを使用して snapshot を構成する方法を示しています。
</VersionBlock>

<VersionBlock lastVersion="1.8">
The following examples demonstrate how to configure snapshots using the `dbt_project.yml` file, a `config` block within a snapshot (legacy method), and a `.yml` file.
</VersionBlock>

- #### すべての snapshot に設定を適用する
  インストール済みの[パッケージ](/docs/build/packages)内の snapshot を含むすべての snapshot に設定を適用するには、設定を`snapshots`キーの直下にネストします。

    <File name='dbt_project.yml'>

    ```yml
    snapshots:
      +unique_key: id
    ```

    </File>

- #### プロジェクト内のすべての snapshot に設定を適用する
  プロジェクト内のすべての snapshot にのみ設定を適用するには（たとえば、インストール済みパッケージ内の snapshot を_除外_する）、リソースパスの一部としてプロジェクト名を指定します。

  `jaffle_shop` というプロジェクトの場合：

    <File name='dbt_project.yml'>

    ```yml
    snapshots:
      jaffle_shop:
        +unique_key: id
    ```

    </File>

  同様に、インストールされたパッケージの名前を使用して、そのパッケージ内の snapshot を構成することもできます。

- #### 1つの snapshot にのみ構成を適用する
  
  <VersionBlock lastVersion="1.8">
  Use `config` blocks if you need to apply a configuration to one snapshot only. 

    <File name='snapshots/postgres_app/orders_snapshot.sql'>

    ```sql
    {% snapshot orders_snapshot %}
        {{
            config(
              unique_key='id',
              target_schema='snapshots',
              strategy='timestamp',
              updated_at='updated_at'
            )
        }}
        -- Pro-Tip: Use sources in snapshots!
        select * from {{ source('jaffle_shop', 'orders') }}
    {% endsnapshot %}
    ```

    </File>
    </VersionBlock>

    <VersionBlock firstVersion="1.9">
     <File name='snapshots/postgres_app/order_snapshot.yml'>

    ```yaml
    snapshots:
     - name: orders_snapshot
       relation: source('jaffle_shop', 'orders')
       config:
         unique_key: id
         strategy: timestamp
         updated_at: updated_at
         persist_docs:
           relation: true
           columns: true
    ```
    </File>
   プロのヒント:  snapshot でソースを使用する: `select * from {{ source('jaffle_shop', 'orders') }}`
    </VersionBlock>

  `dbt_project.yml` ファイルから、完全なリソースパス（プロジェクト名とサブディレクトリを含む）を使用して個別の snapshot を設定することもできます。

  `jaffle_shop` というプロジェクトで、`snapshots/postgres_app/` ディレクトリ内に snapshot ファイルがあり、 snapshot の名前が `orders_snapshot` の場合（上記のように）、設定は次のようになります:

    <File name='dbt_project.yml'>

    ```yml
    snapshots:
      jaffle_shop:
        postgres_app:
          orders_snapshot:
            +unique_key: id
            +strategy: timestamp
            +updated_at: updated_at
    ```

    </File>

   snapshot の `config` ブロックで一般的な設定を定義することもできます。ただし、 snapshot の必須設定にはこれを推奨しません。

    <File name='dbt_project.yml'>

    ```yml
    version: 2

    snapshots:
      - name: orders_snapshot
        +persist_docs:
          relation: true
          columns: true
    ```

    </File>
