---
resource_types: all
datatype: markdown_string
description: "This guide explains how to use the description key to add YAML descriptions to dbt resources (models, sources, seeds) using markdown and Jinja for better documentation."
---

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Sources', value: 'sources', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
    { label: 'Analyses', value: 'analyses', },
    { label: 'Macros', value: 'macros', },
    { label: 'Data tests', value: 'data_tests', },
    { label: 'Unit tests', value: 'unit_tests', },
  ]
}>
<TabItem value="models">

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: model_name
    description: markdown_string

    columns:
      - name: column_name
        description: markdown_string

```

</File>

</TabItem>

<TabItem value="sources">

<File name='models/schema.yml'>

```yml
version: 2

sources:
  - name: source_name
    description: markdown_string

    tables:
      - name: table_name
        description: markdown_string

        columns:
          - name: column_name
            description: markdown_string

```

</File>

</TabItem>

<TabItem value="seeds">

<File name='seeds/schema.yml'>

```yml
version: 2

seeds:
  - name: seed_name
    description: markdown_string

    columns:
      - name: column_name
        description: markdown_string

```

</File>

</TabItem>

<TabItem value="snapshots">

<File name='snapshots/schema.yml'>

```yml
version: 2

snapshots:
  - name: snapshot_name
    description: markdown_string

    columns:
      - name: column_name
        description: markdown_string

```

</File>

</TabItem>

<TabItem value="analyses">

<File name='analysis/schema.yml'>

```yml
version: 2

analyses:
  - name: analysis_name
    description: markdown_string

    columns:
      - name: column_name
        description: markdown_string

```

</File>

</TabItem>

<TabItem value="macros">

<File name='macros/schema.yml'>

```yml
version: 2

macros:
  - name: macro_name
    description: markdown_string

    arguments:
      - name: argument_name
        description: markdown_string

```

</File>

</TabItem>

<TabItem value="data_tests">

<VersionBlock firstVersion="1.9">

[特異データ テスト](/docs/build/data-tests#singular-data-tests) または [汎用データ テスト](/docs/build/data-tests#generic-data-tests) に説明を追加できます。

<File name='tests/schema.yml'>

```yml
# Singular data test example

version: 2

data_tests:
  - name: data_test_name
    description: markdown_string
```
</File>

<File name='tests/schema.yml'>

```yml
# Generic data test example

version: 2

models:
  - name: model_name
    columns:
      - name: column_name
        tests:
          - unique:
              description: markdown_string
```
</File>

</VersionBlock>

<VersionBlock lastVersion="1.8">

The `description` property is available for [singular data tests](/docs/build/data-tests#singular-data-tests) or [generic data tests](/docs/build/data-tests#generic-data-tests) beginning in dbt v1.9.

</VersionBlock> 

</TabItem>

<TabItem value="unit_tests">

<VersionBlock firstVersion="1.8">

<File name='models/schema.yml'>

```yml
unit_tests:
  - name: unit_test_name
    description: "markdown_string"
    model: model_name 
    given: ts
      - input: ref_or_source_call
        rows:
         - {column_name: column_value}
         - {column_name: column_value}
         - {column_name: column_value}
         - {column_name: column_value}
      - input: ref_or_source_call
        format: csv
        rows: dictionary | string
    expect: 
      format: dict | csv | sql
      fixture: fixture_name
```

</File>

</VersionBlock>

<VersionBlock lastVersion="1.7">

The `description` property is available for [unit tests](/docs/build/unit-tests) beginning in dbt v1.8.

</VersionBlock>

</TabItem>

</Tabs>

## 定義

以下の内容を文書化するために使用するユーザー定義の説明:

- a model, and model columns
- sources, source tables, and source columns
- seeds, and seed columns
- snapshots, and snapshot columns
- analyses, and analysis columns
- macros, and macro arguments
- data tests, and data test columns
- unit tests for models

これらの説明は、dbt によって表示されるドキュメントウェブサイトで使用されます（[ドキュメントガイド](/docs/build/documentation) または [dbt Explorer](/docs/collaborate/explore-projects) を参照してください）。

説明には、マークダウンと [`doc` jinja 関数](/reference/dbt-jinja-functions/doc) を使用できます。

:::caution YAMLを引用符で囲む必要がある場合があります

説明を記述する際は、YAMLのセマンティクスに注意してください。説明に中括弧、コロン、角括弧などの特殊なYAML文字が含まれている場合は、説明を引用符で囲む必要がある場合があります。引用符で囲んだ説明の例を[下記](#use-some-markdown-in-a-description)に示します。

:::

## 例

このセクションでは、さまざまなリソースに説明を追加する方法の例を示します。

- [モデルと列に簡単な説明を追加する](#add-a-simple-description-to-a-model-and-column) <br />
- [モデルに複数行の説明を追加する](#add-a-multiline-description-to-a-model) <br />
- [説明にマークダウンを使用する](#use-some-markdown-in-a-description) <br />
- [説明にドキュメントブロックを使用する](#use-a-docs-block-in-a-description) <br />
- [説明に別のモデルへのリンクを追加する](#link-to-another-model-in-a-description)
- [説明にリポジトリから画像を含める](#include-an-image-from-your-repo-in-your-descriptions) <br />
- [説明にウェブ上の画像を含める](#include-an-image-from-the-web-in-your-descriptions) <br />
- [データテストに説明を追加する](#add-a-description-to-a-data-tes) <br />
- [ユニットテストに説明を追加する](#add-a-description-to-a-unit-test) <br />

### Add a simple description to a model and column {#add-a-simple-description-to-a-model-and-column}

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: dim_customers
    description: One record per customer

    columns:
      - name: customer_id
        description: Primary key

```

</File>

### Add a multiline description to a model {#add-a-multiline-description-to-a-model}

YAML [ブロック表記](https://yaml-multiline.info/)を使用すると、長い説明を複数行に分割できます:

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: dim_customers
    description: >
      One record per customer. Note that a customer must have made a purchase to
      be included in this <Term id="table" /> — customer accounts that were created but never
      used have been filtered out.

    columns:
      - name: customer_id
        description: Primary key.

```

</File>

### Use some markdown in a description {#use-some-markdown-in-a-description}

説明にはマークダウンを使用できますが、YAML パーサーが特殊文字によって混乱しないように、説明を引用符で囲む必要がある場合があります。

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: dim_customers
    description: "**\[Read more](https://www.google.com/)**"

    columns:
      - name: customer_id
        description: Primary key.

```

</File>

### Use a docs block in a description {#use-a-docs-block-in-a-description}

説明が長い場合、特にマークダウンが含まれている場合は、[`docs` ブロック](/reference/dbt-jinja-functions/doc)を活用すると効果的です。このアプローチの利点は、コードエディタがマークダウンを正しくハイライト表示するため、記述中のデバッグが容易になることです。

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: fct_orders
    description: This table has basic information about orders, as well as some derived facts based on payments

    columns:
      - name: status
        description: '{{ doc("orders_status") }}'

```

</File>

<File name='models/docs.md'>

```

{% docs orders_status %}

Orders can be one of the following statuses:

| status         | description                                                               |
|----------------|---------------------------------------------------------------------------|
| placed         | The order has been placed but has not yet left the warehouse              |
| shipped        | The order has been shipped to the customer and is currently in transit     |
| completed      | The order has been received by the customer                               |
| returned       | The order has been returned by the customer and received at the warehouse |


{% enddocs %}

```

</File>


### 説明内で別のモデルへのリンク {#link-to-another-model-in-a-description}

相対リンクを使って別のモデルにリンクできます。少し面倒ですが、以下の手順で行います。

1. ドキュメントサイトを開きます。
2. リンク先のモデルに移動します。例: `http://127.0.0.1:8080/#!/model/model.jaffle_shop.stg_stripe__payments`
3. url_path（`http://127.0.0.1:8080/` の後のすべて、つまりこの場合は `#!/model/model.jaffle_shop.stg_stripe__payments`）をコピーします。
4. それをリンクとして貼り付けます。

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: customers
    description: "Filtering done based on \[stg_stripe__payments](#!/model/model.jaffle_shop.stg_stripe__payments)"

    columns:
      - name: customer_id
        description: Primary key

```

</File>


### 説明にリポジトリの画像を含める {#include-an-image-from-your-repo-in-your-description}

このセクションは dbt Core ユーザーのみに適用されます。リポジトリの画像を含めることで、画像のバージョン管理が確実に行われます。

dbt Cloud ユーザーと dbt Core ユーザーの両方が [Web からの画像を含める](#説明に Web からの画像を含める) ことができます。これにより、動的なコンテンツ、リポジトリサイズの削減、アクセシビリティ、そして共同作業の容易さが実現します。

モデルの `description` フィールドに画像を含めるには:

1. サブディレクトリにファイルを追加します (例: `assets/dbt-logo.svg`)。
2. `dbt_project.yml` ファイルの [`asset-paths` 設定](/reference/project-configs/asset-paths) を設定し、`dbt docs generate` の実行時にこのディレクトリが `target/` ディレクトリにコピーされるようにします。

<File name='dbt_project.yml'>

```yml
asset-paths: ["assets"]
```

</File>

2. Use a Markdown link to the image in your `description:`

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: customers
    description: "!\[dbt Logo](assets/dbt-logo.svg)"

    columns:
      - name: customer_id
        description: Primary key

```

</File>

3. `dbt docs generate` を実行します。`assets` ディレクトリが `target` ディレクトリにコピーされます。

4. `dbt docs serve` を実行します。画像がプロジェクトドキュメントの一部としてレンダリングされます。

画像とテキストを混在させる場合は、docs ブロックの使用も検討してください。

### 説明にウェブ上の画像を含める {#include-an-image-from-the-web-in-your-descriptions}

このセクションは、dbt Cloud および dbt Core ユーザーに適用されます。ウェブ上の画像を含めることで、動的なコンテンツ、リポジトリサイズの削減、アクセシビリティの向上、そして共同作業の容易化といったメリットが得られます。

ウェブ上の画像を含めるには、モデルの「description」フィールドに画像の URL を指定します:

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: customers
    description: "!\[dbt Logo](https://github.com/dbt-labs/dbt-core/blob/main/etc/dbt-core.svg)"

    columns:
      - name: customer_id
        description: Primary key

```

</File>

画像とテキストを混在させる場合は、ドキュメント ブロックの使用も検討してください。

### データテストに説明を追加する {#add-a-description-to-a-data-test}

<VersionBlock lastVersion="1.8">

<VersionCallout version="1.9" />

</VersionBlock>

汎用データ テストまたは特異データ テストに `description` プロパティを追加できます。

#### 汎用データテスト

この例は、`orders` モデルの列内の一意の値をチェックする汎用データテストを示しています。

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique:
              description: "The order_id is unique for every row in the orders model"
```
</File>

汎用データテストのコアロジックを提供するJinjaマクロに説明を追加することもできます。詳細については、[汎用データテストのロジックに説明を追加する](/best-practices/writing-custom-generic-tests#add-description-to-generic-data-test-logic)を参照してください。

#### 特異データテスト

この例は、`payments` モデル内のすべての値が負でない (≥ 0) ことを確認する特異データテストを示しています。

<File name='tests/<filename>.yml'>

```yaml
version: 2
data_tests:
  - name: assert_total_payment_amount_is_positive
    description: >
      Refunds have a negative amount, so the total amount should always be >= 0.
      Therefore return records where total amount < 0 to make the test fail.

```
</File>

テストを実行するには、`tests` ディレクトリに `tests/assert_total_payment_amount_is_positive.sql` SQL ファイルが存在している必要があることに注意してください。

### ユニットテストに説明を追加する {#add-a-description-to-a-unit-test}

<VersionBlock lastVersion="1.7">

<VersionCallout version="1.8" />

</VersionBlock>

この例では、`opened_at` タイムスタンプが `stg_locations` モデルの日付に適切に切り捨てられていることを確認する単体テストを示します。

<File name='models/<filename>.yml'>

```yaml
unit_tests:
  - name: test_does_location_opened_at_trunc_to_date
    description: "Check that opened_at timestamp is properly truncated to a date."
    model: stg_locations
    given:
      - input: source('ecom', 'raw_stores')
        rows:
          - {id: 1, name: "Rego Park", tax_rate: 0.2, opened_at: "2016-09-01T00:00:00"}
          - {id: 2, name: "Jamaica", tax_rate: 0.1, opened_at: "2079-10-27T23:59:59.9999"}
    expect:
      rows:
        - {location_id: 1, location_name: "Rego Park", tax_rate: 0.2, opened_date: "2016-09-01"}
        - {location_id: 2, location_name: "Jamaica", tax_rate: 0.1, opened_date: "2079-10-27"}
```

</File>
