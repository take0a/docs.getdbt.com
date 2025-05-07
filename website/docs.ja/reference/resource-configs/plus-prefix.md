---
title: + プレフィックスの使用
description: "+ プレフィックスは、dbt_project.yml ファイル内のリソース パスと構成の区別に役立ちます。"
intro_text: "dbt_project.yml ファイル内のリソース パスと構成の違いを明確にするには、+ プレフィックスを使用します。"
---

`+` プレフィックスは、dbt 構文の機能であり、[リソースパス](/reference/resource-configs/resource-path) と [`dbt_project.yml` ファイル](/reference/dbt_project.yml) 内の設定を区別するのに役立ちます。

- これは、[`config-version`](/reference/project-configs/config-version) 1 を使用する `dbt_project.yml` ファイルとは互換性がありません。
- 以下のファイルには適用されません。
- リソースファイル内の `config()` Jinja マクロ
- `.yml` ファイル内の設定プロパティ

例:

<File name='dbt_project.yml'>

```yml
name: jaffle_shop
config-version: 2

...

models:
  +materialized: view
  jaffle_shop:
    marts:
      +materialized: table
```

</File>

このドキュメント全体を通して、`dbt_project.yml` ファイルでは `+` プレフィックスの使用に一貫性を持たせるように努めてきました。

ただし、先頭の `+` は、リソースパスと設定を区別する必要がある場合にのみ必要です。例えば、以下の場合です。
- 設定が入力として辞書を受け入れる場合。例として、[`persist_docs` 設定](/reference/resource-configs/persist_docs) が挙げられます。
- または、設定がリソースパスの一部とキーを共有している場合。例えば、`tags` という名前のモデルのディレクトリがある場合などです。

<File name='dbt_project.yml'>

```yml
name: jaffle_shop
config-version: 2

...

models:
  +persist_docs: # this config is a dictionary, so needs a + prefix
    relation: true
    columns: true

  jaffle_shop:
    schema: my_schema # a plus prefix is optional here
    +tags: # this is the tag config
      - "hello"
    tags: # whereas this is the tag resource path
      # The below config applies to models in the
      # models/tags/ directory.
      # Note: you don't _need_ a leading + here,
      # but it wouldn't hurt.
      materialized: view


```

</File>

`dbt_project.yml` に構成を追加する際は、`+` プレフィックスを使用しても問題はありませんので、常に使用することをお勧めします。

**注:** `dbt_project.yml` におけるこの `+` プレフィックスの使用は、他の構成設定（特定のリソース `.yml` および `.sql` ファイル）における構成のマージ動作（上書きまたは追加）を制御するための `+` の使用とは異なります。現在、構成のマージ動作を制御するために `+` をサポートしている構成は [`grants`](/reference/resource-configs/grants#grant-config-inheritance) のみです。

