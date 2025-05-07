---
resource_types: [models, seeds]
description: "dbt 内のモデルやその他のリソースの full_refresh 構成を設定します。"
datatype: boolean
---

`full_refresh` 設定を使用すると、リソースが常にフルリフレッシュを実行するか、あるいは実行しないかを制御できます。この設定は `--full-refresh` コマンドラインフラグをオーバーライドします。

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Seeds', value: 'seeds', },
  ]
}>

<TabItem value="models">

<File name='dbt_project.yml'>

```yml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +full_refresh: false | true 
```

</File>

<File name='models/<modelname>.sql'>

```sql

{{ config(
    full_refresh = false | true
) }}

select ...
```

</File>

</TabItem>

<TabItem value="seeds">

<File name='dbt_project.yml'>

```yml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    +full_refresh: false | true

```

</File>

</TabItem>

</Tabs>

## 説明

`full_refresh` 設定を使用すると、リソースが常にフルリフレッシュを実行するか、あるいは実行しないかをオプションで設定できます。この設定は、dbt コマンド実行時に使用される `--full-refresh` コマンドラインフラグをオーバーライドします。

`full_refresh` 設定は、`dbt_project.yml` ファイルまたはリソース設定で設定できます。

| `full_refresh` value | Behavior |
| ---------------------------- | -------- |
| If set to `true` | dbt コマンドで `--full-refresh` フラグを渡すかどうかに関係なく、リソースは常に完全更新を実行します。 |
| If set to `false` | dbt コマンドで `--full-refresh` フラグを渡すかどうかに関係なく、リソースは完全な更新を決して実行しません。 |
| If set to `none` or omitted | リソースは `--full-refresh` フラグの動作に従います。このフラグが使用されている場合、リソースは完全リフレッシュを実行します。そうでない場合は、リフレッシュは実行されません。 |

#### 注
- `--full-refresh` フラグは、短縮名 `-f` もサポートしています。
- [`should_full_refresh()`](https://github.com/dbt-labs/dbt-adapters/blob/60005a0a2bd33b61cb65a591bc1604b1b3fd25d5/dbt/include/global_project/macros/materializations/configs.sql) マクロにはロジックがエンコードされています。

## 使用法

### 増分モデル

* [増分モデルを再構築するにはどうすればよいですか？](/docs/build/incremental-models#how-do-i-rebuild-an-incremental-model)
* [増分モデルの列が変更された場合はどうなりますか？](/docs/build/incremental-models#what-if-the-columns-of-my-incremental-model-change)

### Seeds

<FAQ path="Seeds/full-refresh-seed" />

## 推奨事項
特に大規模なデータセットのモデルでは、dbt で完全に削除して再作成する必要がないため、`full_refresh: false` を設定してください。

## リファレンスドキュメント
* [on_configuration_change](/reference/resource-configs/on_configuration_change)
