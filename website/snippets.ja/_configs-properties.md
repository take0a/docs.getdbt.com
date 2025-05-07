
## 例
以下は、プロジェクトの `sources` と `models` の両方を定義する例です。

<File name='models/jaffle_shop.yml'>

```yml
version: 2

sources:
  - name: raw_jaffle_shop
    description: A replica of the postgres database used to power the jaffle_shop app.
    tables:
      - name: customers
        columns:
          - name: id
            description: Primary key of the table
            tests:
              - unique
              - not_null

      - name: orders
        columns:
          - name: id
            description: Primary key of the table
            tests:
              - unique
              - not_null

          - name: user_id
            description: Foreign key to customers

          - name: status
            tests:
              - accepted_values:
                  values: ['placed', 'shipped', 'completed', 'return_pending', 'returned']


models:
  - name: stg_jaffle_shop__customers #  Must match the filename of a model -- including case sensitivity.
    config:
      tags: ['pii']
    columns:
      - name: customer_id
        tests:
          - unique
          - not_null

  - name: stg_jaffle_shop__orders
    config:
      materialized: view
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'return_pending', 'returned']
              config:
                severity: warn


```

</File>

## 関連ドキュメント
サポートされている各プロパティと設定の詳細なリストは、リソースタイプ別に記載されています。
* Model [properties](/reference/model-properties) and [configs](/reference/model-configs)
* Source [properties](/reference/source-properties) and [configs](source-configs)
* Seed [properties](/reference/seed-properties) and [configs](/reference/seed-configs)
* Snapshot [properties](snapshot-properties)
* Analysis [properties](analysis-properties)
* Macro [properties](/reference/macro-properties)
* Exposure [properties](/reference/exposure-properties)

## FAQs
<FAQ path="Project/schema-yml-name" />
<FAQ path="Project/resource-yml-name" />
<FAQ path="Project/multiple-resource-yml-files" />
<FAQ path="Project/properties-not-in-config" />
<FAQ path="Project/why-version-2" />
<FAQ path="Project/yaml-file-extension" />


## Troubleshooting common errors

<Expandable alt_header="Invalid test config given in [model name]">

このエラーは、`.yml` ファイルが dbt が想定する構造に準拠していない場合に発生します。完全なエラーメッセージは次のようになります:

```
* Invalid test config given in models/schema.yml near {'namee': 'event', ...}
  Invalid arguments passed to "UnparsedNodeUpdate" instance: 'name' is a required property, Additional properties are not allowed ('namee' was unexpected)
```

このようなエラーは冗長ではありますが、問題の追跡に役立つはずです。ここでは、`name` フィールドが誤って `namee` として指定されています。このエラーを修正するには、`.yml` がこのガイドに記載されている想定される構造に準拠していることを確認してください。

</Expandable>

<Expandable alt_header="schema.yml ファイルの構文が無効です" >

`.yml` ファイルが有効な yaml でない場合、dbt は次のようなエラーを表示します。

```text
Runtime Error
  Syntax error near line 6
  ------------------------------
  5  |   - name: events
  6  |     description; "A table containing clickstream events from the marketing website"
  7  |

  Raw Error:
  ------------------------------
  while scanning a simple key
    in "<unicode string>", line 6, column 5:
          description; "A table containing clickstream events from the marketing website"
          ^

```

このエラーは、`description` フィールドの後にコロン (`:`) の代わりにセミコロン (`;`) が誤って使用されたために発生しました。このような問題を解決するには、エラーメッセージで参照されている `.yml` ファイルを見つけ、ファイル内の構文エラーを修正してください。オンラインの YAML バリデーターが役立ちますが、サードパーティのアプリケーションに機密情報を送信する際にはご注意ください。

</Expandable>
