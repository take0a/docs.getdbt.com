---
datatype: [directorypath]
description: "dbt の snapshot-paths の構成を理解するには、このガイドをお読みください。"
default_value: [snapshots]
---
<File name='dbt_project.yml'>

```yml
snapshot-paths: [directorypath]
```

</File>

## 定義

オプションで、[スナップショット](/docs/build/snapshots)が配置されているディレクトリのカスタムリストを指定します。

<VersionBlock firstVersion="1.9">
dbt Core v1.9 以降では、[最新の YAML 構文を使用して定義されている](/docs/build/snapshots) 場合、スナップショットをモデルと同じ場所に配置できます。
</VersionBlock>

<VersionBlock lastVersion="1.8">
Note that you cannot co-locate models and snapshots. However, in dbt Core v1.9+, you can co-locate your snapshots with models if they are [defined using the latest YAML syntax](/docs/build/snapshots).
</VersionBlock>

## デフォルト

デフォルトでは、dbt は `snapshots` ディレクトリ内のスナップショットを検索します。たとえば、`snapshot-paths: ["snapshots"]` のようになります。

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="snapshot-paths"
absolute="/Users/username/project/snapshots"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    snapshot-paths: ["snapshots"]
    ```

- ❌ **Don't:**
  - 絶対パスは避けてください:
    ```yml
    snapshot-paths: ["/Users/username/project/snapshots"]
    ```

## 例
### `snapshots` の代わりに `archives` という名前のサブディレクトリを使用します

<File name='dbt_project.yml'>

```yml
snapshot-paths: ["archives"]
```

</File>
