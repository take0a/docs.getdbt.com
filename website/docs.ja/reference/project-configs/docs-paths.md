---
datatype: [directorypath]
description: "dbt の docs-paths 構成を理解するには、このガイドをお読みください。"
default_value: []
---

<File name='dbt_project.yml'>

```yml
docs-paths: [directorypath]
```

</File>

## 定義
オプションで、[ドキュメントブロック](/docs/build/documentation#docs-blocks)が配置されているディレクトリのカスタムリストを指定します。


## デフォルト

<VersionBlock firstVersion="1.9">

デフォルトでは、dbt はすべてのリソース パスで docs ブロックを検索します (たとえば、[model-paths](/reference/project-configs/model-paths)、[seed-paths](/reference/project-configs/seed-paths)、[analysis-paths](/reference/project-configs/analysis-paths)、[test-paths](/reference/project-configs/test-paths)、[macro-paths](/reference/project-configs/macro-paths)、および [snapshot-paths](/reference/project-configs/snapshot-paths) を組み合わせたリスト)。このオプションを構成すると、dbt は指定されたディレクトリでのみ docs ブロックを検索します。

</VersionBlock>

<VersionBlock lastVersion="1.8">

By default, dbt will search in all resource paths for docs blocks (i.e. the combined list of [model-paths](/reference/project-configs/model-paths), [seed-paths](/reference/project-configs/seed-paths), [analysis-paths](/reference/project-configs/analysis-paths), [macro-paths](/reference/project-configs/macro-paths), and [snapshot-paths](/reference/project-configs/snapshot-paths)). If this option is configured, dbt will _only_ look in the specified directory for docs blocks.

</VersionBlock>

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="docs-paths"
absolute="/Users/username/project/docs"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    docs-paths: ["docs"]
    ```

- ❌ **Don't**
  - 絶対パスは避けてください:
    ```yml
    docs-paths: ["/Users/username/project/docs"]
    ```

## 例

docs ブロックには `docs` というサブディレクトリを使用します:

<File name='dbt_project.yml'>

```yml
docs-paths: ["docs"]
```

</File>

**注:** dbt のデフォルトの動作を優先するため、通常はこの構成を省略します。