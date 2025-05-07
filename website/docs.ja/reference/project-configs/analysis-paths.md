---
datatype: [directorypath]
description: "dbt の analysis-paths 構成を理解するには、このガイドをお読みください。"
default_value: []
---

<File name='dbt_project.yml'>

```yml
analysis-paths: [directorypath]
```

</File>

## 定義
[analyses](/docs/build/analyses) が配置されているディレクトリのカスタムリストを指定します。

## デフォルト
この設定を指定しないと、dbt は `.sql` ファイルを analyses としてコンパイルしません。

ただし、[`dbt init` コマンド](/reference/commands/init) は、この値を `analyses` として入力します ([ソース](https://github.com/dbt-labs/dbt-starter-project/blob/HEAD/dbt_project.yml#L15))。

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="analysis-paths"
absolute="/Users/username/project/analyses"
/>

- ✅ **Do** 
  - 相対パスを使用:
    ```yml
    analysis-paths: ["analyses"]
    ```

- ❌ **Don't** 
  - 絶対パスは避けてください:
    ```yml
    analysis-paths: ["/Users/username/project/analyses"]
    ```

## 例
### `analyses`という名前のサブディレクトリを使用する
これは、[`dbt init` コマンド](/reference/commands/init) によって設定される値です。

<File name='dbt_project.yml'>

```yml
analysis-paths: ["analyses"]
```

</File>

### `custom_analyses`という名前のサブディレクトリを使用します

<File name='dbt_project.yml'>

```yml
analysis-paths: ["custom_analyses"]
```

</File>
