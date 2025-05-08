---
title: Analysis プロパティ
---

import PropsCallout from '/snippets.ja/_config-prop-callout.md';

analysis プロパティは、`analyses/` ディレクトリで定義することをお勧めします。これは、[`analysis-paths`](/reference/project-configs/analysis-paths) 構成で示されています。<PropsCallout title={frontMatter.title}/> <br />

これらのファイルには `whatever_you_want.yml` という名前を付け、`analyses/` または `models/` ディレクトリ内のサブフォルダに任意の深さでネストできます。

<File name='analyses/<filename>.yml'>

```yml
version: 2

analyses:
  - name: <analysis_name> # required
    [description](/reference/resource-properties/description): <markdown_string>
    [docs](/reference/resource-configs/docs):
      show: true | false
      node_color: <color_id> # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
    config:
      [tags](/reference/resource-configs/tags): <string> | [<string>]
    columns:
      - name: <column_name>
        [description](/reference/resource-properties/description): <markdown_string>
      - name: ... # declare properties of additional columns

  - name: ... # declare properties of additional analyses

```

</File>
