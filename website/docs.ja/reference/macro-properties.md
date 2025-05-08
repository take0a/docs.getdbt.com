---
title: Macro プロパティ
id: macro-properties
---

import PropsCallout from '/snippets.ja/_config-prop-callout.md';

マクロプロパティは、任意の `properties.yml` ファイルで宣言できます。<PropsCallout title={frontMatter.title}/>

これらのファイルには `whatever_you_want.yml` という名前を付け、任意の深さのサブフォルダにネストできます。

<File name='macros/<filename>.yml'>

```yml
version: 2

macros:
  - name: <macro name>
    [description](/reference/resource-properties/description): <markdown_string>
    [docs](/reference/resource-configs/docs):
      show: true | false
    [meta](/reference/resource-configs/meta): {<dictionary>}
    arguments:
      - name: <arg name>
        [type](/reference/resource-properties/argument-type): <string>
        [description](/reference/resource-properties/description): <markdown_string>
      - ... # declare properties of additional arguments

  - name: ... # declare properties of additional macros

```

</File>
