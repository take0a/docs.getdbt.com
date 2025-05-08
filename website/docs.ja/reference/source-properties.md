---
title: "Source プロパティ"
description: "dbt で source プロパティを使用する方法を学習します。"
---

## 関連ドキュメント
- [ソースの使用](/docs/build/sources)
- [リソースプロパティの宣言](/reference/configs-and-properties)

## 概要

import PropsCallout from '/snippets/_config-prop-callout.md';

ソースプロパティは、`models/` ディレクトリ内の任意の `properties.yml` ファイルで宣言できます（[`model-paths` 設定](/reference/project-configs/model-paths) で定義されています）。<PropsCallout title={frontMatter.title}/> <br />

これらのファイルの名前を `whatever_you_want.yml` とし、`models/` ディレクトリ内の任意の階層にサブフォルダとしてネストできます。

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <string> # required
    [description](/reference/resource-properties/description): <markdown_string>
    [database](/reference/resource-properties/database): <database_name>
    [schema](/reference/resource-properties/schema): <schema_name>
    [loader](/reference/resource-properties/loader): <string>
    [loaded_at_field](/reference/resource-properties/freshness#loaded_at_field): <column_name>
    [meta](/reference/resource-configs/meta): {<dictionary>}
    [tags](/reference/resource-configs/tags): [<string>]
    
    # requires v1.1+
    [config](/reference/resource-properties/config):
      [<source_config>](source-configs): <config_value>

    [overrides](/reference/resource-properties/overrides): <string>

    [freshness](/reference/resource-properties/freshness):
      warn_after:
        [count](/reference/resource-properties/freshness#count): <positive_integer>
        [period](/reference/resource-properties/freshness#period): minute | hour | day
      error_after:
        [count](/reference/resource-properties/freshness#count): <positive_integer>
        [period](/reference/resource-properties/freshness#period): minute | hour | day
      [filter](/reference/resource-properties/freshness#filter): <where-condition>

    [quoting](/reference/resource-properties/quoting):
      database: true | false
      schema: true | false
      identifier: true | false

    tables:
      - name: <string> #required
        [description](/reference/resource-properties/description): <markdown_string>
        [meta](/reference/resource-configs/meta): {<dictionary>}
        [identifier](/reference/resource-properties/identifier): <table_name>
        [loaded_at_field](/reference/resource-properties/freshness#loaded_at_field): <column_name>
        [tests](/reference/resource-properties/data-tests):
          - <test>
          - ... # declare additional tests
        [tags](/reference/resource-configs/tags): [<string>]
        [freshness](/reference/resource-properties/freshness):
          warn_after:
            [count](/reference/resource-properties/freshness#count): <positive_integer>
            [period](/reference/resource-properties/freshness#period): minute | hour | day
          error_after:
            [count](/reference/resource-properties/freshness#count): <positive_integer>
            [period](/reference/resource-properties/freshness#period): minute | hour | day
          [filter](/reference/resource-properties/freshness#filter): <where-condition>

        [quoting](/reference/resource-properties/quoting):
          database: true | false
          schema: true | false
          identifier: true | false
        [external](/reference/resource-properties/external): {<dictionary>}
        columns:
          - name: <column_name> # required
            [description](/reference/resource-properties/description): <markdown_string>
            [meta](/reference/resource-configs/meta): {<dictionary>}
            [quote](/reference/resource-properties/columns#quote): true | false
            [tests](/reference/resource-properties/data-tests):
              - <test>
              - ... # declare additional tests
            [tags](/reference/resource-configs/tags): [<string>]
          - name: ... # declare properties of additional columns

      - name: ... # declare properties of additional source tables

  - name: ... # declare properties of additional sources

```

</File>


## Example

<File name='models/<filename>.yml'>

```yaml
version: 2

sources:
  - name: jaffle_shop
    database: raw
    schema: public
    loader: emr # informational only (free text)
    loaded_at_field: _loaded_at # configure for all sources

    # meta fields are rendered in auto-generated documentation
    meta:
      contains_pii: true
      owner: "@alice"

    # Add tags to this source
    tags:
      - ecom
      - pii

    quoting:
      database: false
      schema: false
      identifier: false

    tables:
      - name: orders
        identifier: Orders_
        loaded_at_field: updated_at # override source defaults
        columns:
          - name: id
            tests:
              - unique

          - name: price_in_usd
            tests:
              - not_null

      - name: customers
        quoting:
          identifier: true # override source defaults
        columns:
            tests:
              - unique
```

</File>
