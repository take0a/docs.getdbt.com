---
resource_types: [models]
description: "on_configuration_change - dbt での構成変更監視について詳しくは、この詳細なガイドをお読みください。"
datatype: "string"
---

:::info
この機能は現在、アダプタのサブセット上の [マテリアライズド ビュー](/docs/build/materializations#materialized-view) に対してのみサポートされています。
:::

`on_configuration_change` 構成には、次の 3 つの設定があります。
- `apply` (デフォルト) - 可能な場合は、既存のデータベースオブジェクトの更新を試行し、完全な再構築を回避します。
  - *注:* 個々の構成変更で完全な更新が必要な場合は、個々の変更ステートメントの代わりに完全な更新が実行されます。
- `continue` - 実行を続行しますが、オブジェクトが変更されなかったことを示す警告も表示します。
  - *注:* 実装されていない変更がモデルに適用される可能性があるため、下流でエラーが発生する可能性があります。
- `fail` - 変更が検出された場合、実行全体を強制的に失敗させます。

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
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[materialized](/reference/resource-configs/materialized): <materialization_name>
    [+](/reference/resource-configs/plus-prefix)on_configuration_change: apply | continue | fail
```

</File>

</TabItem>


<TabItem value="property-yaml">

<File name='models/properties.yml'>

```yaml
version: 2

models:
  - name: [<model-name>]
    config:
      [materialized](/reference/resource-configs/materialized): <materialization_name>
      on_configuration_change: apply | continue | fail
```

</File>

</TabItem>


<TabItem value="config">

<File name='models/<model_name>.sql'>

```jinja
{{ config(
    [materialized](/reference/resource-configs/materialized)="<materialization_name>",
    on_configuration_change="apply" | "continue" | "fail"
) }}
```

</File>

</TabItem>

</Tabs>

マテリアライゼーションは、次の「ドロップスルー」ライフサイクルに従って実装されます。
1. 指定されたパスのモデルが存在しない場合は、新しいモデルを作成します。
2. モデルは存在するがタイプが異なる場合は、既存のモデルを削除して新しいモデルを作成します。
3. [`--full-refresh`](/reference/resource-configs/full_refresh) が指定されている場合は、構成の変更や `on_configuration_change` 設定に関係なく、既存のモデルを置き換えます。
4. 構成の変更がない場合は、そのタイプのデフォルトのアクションを実行します（例：マテリアライズドビューの場合は更新を適用します）。
5. `on_configuration_change` 設定に従って、構成の変更を適用するかどうかを決定します。
