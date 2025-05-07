---
resource_types: [models]
datatype: latest_version
required: no
---

<File name='models/<schema>.yml'>

```yml
models:
  - name: model_name
    latest_version: 2
    [versions](/reference/resource-properties/versions):
      - v: 2
      - v: 1
```

</File>

## 定義

このモデルの最新バージョン。「最新」バージョンは、以下の場合に使用されます。
1. このモデルに対する「固定されていない」（バージョンが明示的に指定されていない）`ref()` 呼び出しの解決
2. [`version:` 選択方法](/reference/node-selection/methods#version) を使用して、指定されたモデルバージョンが `latest`、`prerelease`、または `old` のいずれであるかに基づいて、モデルバージョンを選択する

この値は、文字列または数値（整数または浮動小数点数）です。このモデルの `versions` リストで指定されている [バージョン識別子](/reference/resource-properties/versions#v) のいずれかである必要があります。

モデルの最新バージョンを実行するには、[`--select` フラグ](/reference/node-selection/syntax) を使用します。詳細と構文については、[モデル バージョン](/docs/collaborate/govern/model-versions#run-a-model-with-multiple-versions) を参照してください。

## デフォルト

バージョン管理されたモデルで指定されていない場合、`latest_version` はデフォルトで最大の[バージョン識別子](/reference/resource-properties/versions#v) になります。すべてのバージョン識別子が数値の場合は数値順に最大、それ以外の場合はアルファベット順で最後になります（文字列の場合）。

バージョン管理されていないモデル（`versions` リストがない）の場合、`latest_version` には値がありません。

バージョン管理されたモデルで `latest_version` が指定されていない場合、`latest_version` はデフォルトで最大のものになります。


## 例

<File name='models/<schema>.yml'>

```yml
models:
  - name: model_name
    [versions](/reference/resource-properties/versions):
      - v: 3
      - v: 2
      - v: 1
```

</File>

`latest_version` が指定されていない場合、`latest_version` は `3` になります。固定されていない参照（`ref('model_name')`）は `model_name.v3` に解決されます。`v1` と `v2` はどちらも「古い」バージョンとみなされます。

<File name='models/<schema>.yml'>

```yml
models:
  - name: model_name
    latest_version: 2
    [versions](/reference/resource-properties/versions):
      - v: 3
      - v: 2
      - v: 1
```

</File>

この場合、`latest_version` は明示的に `2` に設定されています。固定されていない参照はすべて `model_name.v2` に解決されます。`v3` は「プレリリース」、`v1` は「古い」と見なされます。
