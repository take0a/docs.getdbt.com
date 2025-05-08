---
resource_types: [models]
description: "Materialized - dbt でのマテリアライゼーションについて詳しくは、この詳細なガイドをお読みください。"
datatype: "string"
---

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Property file', value: 'property-yaml', },
    { label: 'Config block', value: 'config', },
  ]
}>


<TabItem value="project-yaml">

<File name='dbt_project.yml'>

```yaml
[config-version](/reference/project-configs/config-version): 2

models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +materialized: [<materialization_name>](https://docs.getdbt.com/docs/build/materializations#materializations)
```

</File>

</TabItem>


<TabItem value="property-yaml">

<File name='models/properties.yml'>

```yaml
version: 2

models:
  - name: <model_name>
    config:
      materialized: [<materialization_name>](https://docs.getdbt.com/docs/build/materializations#materializations)

```

</File>

</TabItem>


<TabItem value="config">

<File name='models/<model_name>.sql'>

```jinja
{{ config(
  materialized="[<materialization_name>](https://docs.getdbt.com/docs/build/materializations#materializations)"
) }}

select ...
```

</File>

</TabItem>

</Tabs>

## 定義

[マテリアライゼーション](/docs/build/materializations#materializations) は、dbt モデルをウェアハウスに永続化するための戦略です。dbt には以下のマテリアライゼーション タイプが組み込まれています。

- `ephemeral` - [ephemeral](/docs/build/materializations#ephemeral) モデルはデータベースに直接構築されません。
- `table` - モデルは実行ごとに [table](/docs/build/materializations#table) として再構築されます。
- `view` - モデルは実行ごとに [view](/docs/build/materializations#view) として再構築されます。
- `materialized_view` - ターゲット データベースに [マテリアライズド ビュー](/docs/build/materializations#materialized-view) を作成および管理できます。
- `incremental` - [増分](/docs/build/materializations#incremental)モデルを使用すると、dbtは前回のモデル実行以降にテーブルにレコードを挿入または更新できます。

dbtでは[カスタムマテリアライゼーション](/guides/create-new-materializations?step=1)も設定できます。カスタムマテリアライゼーションは、特定のニーズに合わせてdbtの機能を拡張する強力な手段です。

## 作成の優先順位

<!-- This text is copied from /reference/resource-configs/on_configuration_change.md -->
マテリアライズは、以下の「ドロップスルー」ライフサイクルに従って実装されます。

1. 指定されたパスのモデルが存在しない場合は、新しいモデルを作成します。
2. モデルは存在するがタイプが異なる場合は、既存のモデルを削除して新しいモデルを作成します。
3. [`--full-refresh`](/reference/resource-configs/full_refresh) が指定されている場合は、設定変更や [`on_configuration_change`](/reference/resource-configs/on_configuration_change) の設定に関係なく、既存のモデルを置き換えます。
4. 設定変更がない場合は、そのタイプのデフォルトのアクションを実行します（例：マテリアライズドビューの場合は更新を適用します）。
5. `on_configuration_change` 設定に従って、設定変更を適用するかどうかを決定します。

