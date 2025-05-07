---
title: on-run-start と on-run-end
description: "このガイドを読んで、dbt の on-run-start および on-run-end 構成を理解してください。"
datatype: sql-statement | [sql-statement]
---

import OnRunCommands from '/snippets/_onrunstart-onrunend-commands.md';

<File name='dbt_project.yml'>

```yml
on-run-start: sql-statement | [sql-statement]
on-run-end: sql-statement | [sql-statement]
```

</File>


## 定義

以下のコマンドの開始時または終了時に実行されるSQL文（またはSQL文のリスト）: <OnRunCommands />

`on-run-start`フックと`on-run-end`フックは、SQL文を返す[マクロを呼び出す](#権限付与のためのマクロ呼び出し)こともできます。

## 使用上の注意
* `on-run-end` フックには、コンテキストで使用できる追加の Jinja 変数があります。[ドキュメント](/reference/dbt-jinja-functions/on-run-end-context) をご覧ください。

## 例

### 実行終了時に dbt が使用するすべてのスキーマに対する権限を付与します。
これは、`on-run-end` フックでのみ使用可能な [schemas](/reference/dbt-jinja-functions/schemas) 変数を活用します。

<File name='dbt_project.yml'>

```yml
on-run-end:
  - "{% for schema in schemas %}grant usage on schema {{ schema }} to group reporter; {% endfor %}"

```

</File>

### 権限を付与するためのマクロを呼び出す

<File name='dbt_project.yml'>

```yml
on-run-end: "{{ grant_select(schemas) }}"

```

</File>

### 追加の例
より詳細な例を[こちら](/docs/build/hooks-operations#additional-examples)にまとめました。
