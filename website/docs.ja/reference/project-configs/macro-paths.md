---
datatype: directorypath
description: "dbt の macro-paths 構成を理解するには、このガイドをお読みください。"
default_value: [macros]
---

<File name='dbt_project.yml'>

```yml
macro-paths: [directorypath]
```

</File>

## 定義
オプションで、[マクロ](/docs/build/jinja-macros#macros)が配置されているディレクトリのカスタムリストを指定します。モデルとマクロを同じディレクトリに配置することはできないことに注意してください。

## デフォルト
デフォルトでは、dbt は `macros` という名前のディレクトリ内でマクロを検索します。たとえば、`macro-paths: ["macros"]` のようになります。

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="macro-paths"
absolute="/Users/username/project/macros"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    macro-paths: ["macros"]
    ```

- ❌ **Don't:**
  - 絶対パスは避けてください:
    ```yml
    macro-paths: ["/Users/username/project/macros"]
    ```

## 例
### `macros` の代わりに `custom_macros` という名前のサブディレクトリを使用します

<File name='dbt_project.yml'>

```yml
macro-paths: ["custom_macros"]
```

</File>
