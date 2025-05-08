---
resource_types: [seeds]
description: "Quote_columns - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
datatype: boolean
default_value: false
---

## 定義

オプションの seed 設定。<Term id="table" /> の作成時に、 seed ファイル内の列名を引用符で囲むかどうかを決定します。

* `True` の場合、dbt は seed 用のテーブルを作成する際に、 seed ファイルで定義された列名を引用符で囲み、大文字と小文字の区別を維持します。
* `False` の場合、dbt は seed ファイルで定義された列名を引用符で囲みません。
* 設定されていない場合、列名を引用符で囲むかどうかはアダプタによって異なります。

## 使用法

### すべての seed 列をグローバルに引用符で囲む

<File name='dbt_project.yml'>

```yml
seeds:
  +quote_columns: true
```

</File>

### `seeds/mappings` ディレクトリ内の seeds のみを引用符で囲んでください。

以下のプロジェクトの場合:
* `dbt_project.yml` ファイル内の `name: jaffle_shop`
* `dbt_project.yml` ファイル内の `seed-paths: ["seeds"]`

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    mappings:
      +quote_columns: true
```

</File>

Or:

<File name='seeds/properties.yml'>

```yml
version: 2

seeds:
  - name: mappings
    config:
      quote_columns: true
```

</File>

## 推奨設定

* seed ファイルを使用する場合は、この値を明示的に設定してください。
* 設定は、個々のプロジェクト/ seed ではなく、グローバルに適用してください。
* 列名に特殊文字が含まれている場合、または大文字と小文字の区別を維持する必要がない限り、`quote_columns: false` を設定してください。その場合は、seed 列の名前を変更することを検討してください（これにより、下流のコードが簡素化されます）。
