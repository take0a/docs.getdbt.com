---
title: "data tests プロパティについて"
description: "dbt のデータ テストに使用できるリソース プロパティのリファレンス ガイド。"
sidebar_label: "Data tests"
resource_types: all
datatype: data-test
keywords: [test, tests, custom tests, custom test name, test name]
---

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Sources', value: 'sources', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
    { label: 'Analyses', value: 'analyses', },
  ]
}>

<TabItem value="models">

<File name='models/<filename>.yml'>

```yml
version: 2

models:
  - name: <model_name>
    tests:
      - [<test_name>](#custom-data-test-name):
          <argument_name>: <argument_value>
          [config](/reference/resource-properties/config):
            [<test_config>](/reference/data-test-configs): <config-value>

    [columns](/reference/resource-properties/columns):
      - name: <column_name>
        tests:
          - [<test_name>](#custom-data-test-name)
          - [<test_name>](#custom-data-test-name):
              <argument_name>: <argument_value>
              [config](/reference/resource-properties/config):
                [<test_config>](/reference/data-test-configs): <config-value>
```

</File>

</TabItem>

<TabItem value="sources">

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <source_name>
    tables:
    - name: <table_name>
      tests:
        - [<test_name>](#custom-data-test-name)
        - [<test_name>](#custom-data-test-name):
            <argument_name>: <argument_value>
            [config](/reference/resource-properties/config):
              [<test_config>](/reference/data-test-configs): <config-value>

      columns:
        - name: <column_name>
          tests:
            - [<test_name>](#custom-data-test-name)
            - [<test_name>](#custom-data-test-name):
                <argument_name>: <argument_value>
                [config](/reference/resource-properties/config):
                  [<test_config>](/reference/data-test-configs): <config-value>

```

</File>

</TabItem>

<TabItem value="seeds">

<File name='seeds/<filename>.yml'>

```yml
version: 2

seeds:
  - name: <seed_name>
    tests:
      - [<test_name>](#custom-data-test-name)
      - [<test_name>](#custom-data-test-name):
          <argument_name>: <argument_value>
          [config](/reference/resource-properties/config):
            [<test_config>](/reference/data-test-configs): <config-value>

    columns:
      - name: <column_name>
        tests:
          - [<test_name>](#custom-data-test-name)
          - [<test_name>](#custom-data-test-name):
              <argument_name>: <argument_value>
              [config](/reference/resource-properties/config):
                [<test_config>](/reference/data-test-configs): <config-value>

```

</File>

</TabItem>

<TabItem value="snapshots">

<File name='snapshots/<filename>.yml'>

```yml
version: 2

snapshots:
  - name: <snapshot_name>
    tests:
      - [<test_name>](#custom-data-test-name)
      - [<test_name>](#custom-data-test-name):
          <argument_name>: <argument_value>
          [config](/reference/resource-properties/config):
            [<test_config>](/reference/data-test-configs): <config-value>

    columns:
      - name: <column_name>
        tests:
          - [<test_name>](#custom-data-test-name)
          - [<test_name>](#custom-data-test-name):
              <argument_name>: <argument_value>
              [config](/reference/resource-properties/config):
                [<test_config>](/reference/data-test-configs): <config-value>

```

</File>

</TabItem>


<TabItem value="analyses">

この機能は analyses には実装されていません。

</TabItem>

</Tabs>

## 関連ドキュメント

* [データテストガイド](/docs/build/data-tests)

## 説明

データ `tests` プロパティは、列、<Term id="table" />、または <Term id="view" /> に関するアサーションを定義します。このプロパティには、名前で参照される [汎用テスト](/docs/build/data-tests#generic-data-tests) のリストが含まれます。これには、dbt で利用可能な 4 つの組み込み汎用テストを含めることができます。たとえば、列に重複がなく、null 値がゼロであることを確認するテストを追加できます。これらのテストに渡す引数または [構成](/reference/data-test-configs) は、テスト名の下にネストする必要があります。

これらのテストを定義したら、`dbt test` を実行してその正確性を検証できます。

## すぐに使えるデータテスト

dbt をご利用のすべてのユーザーがすぐに使用できる、4 つの汎用データテストが用意されています。

### `not_null`

このテストは、列に `null` 値が存在しないことを検証します。

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - not_null
```

</File>

### `unique`

このテストは、フィールドに重複した値がないことを検証します。

config と where 句はオプションです。

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique:
              config:
                where: "order_id > 21"
```

</File>

### `accepted_values`

このテストは、列内の「null」以外のすべての値が、指定された「values」リストに存在するかどうかを検証します。リストに指定されていない値が存在する場合、テストは失敗します。

「accepted_values」テストはオプションの「quote」パラメータをサポートしており、デフォルトでは、テストクエリで受け入れられる値のリストをシングルクォーテーションで囲みます。文字列以外の値（整数やブール値など）をテストするには、「quote」設定を明示的に「false」に設定してください。

<File name='schema.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: status
        tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']

      - name: status_id
        tests:
          - accepted_values:
              values: [1, 2, 3, 4]
              quote: false
```

</File>

### `relationships`

このテストは、子テーブル <Term id="table" /> 内のすべてのレコードが、親テーブル内に対応するレコードを持つかどうかを検証します。この特性は「参照整合性」と呼ばれます。

次の例では、すべての注文の `customer_id` が有効な `customer` にマッピングされているかどうかをテストします。

<File name='schema.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: customer_id
        tests:
          - relationships:
              to: ref('customers')
              field: id
```

</File>

`to` 引数は [Relation](/reference/dbt-classes#relation) を受け入れます。つまり、モデルへの `ref` (例: `ref('customers')`)、または `source` (例: `source('jaffle_shop', 'customers')`) を渡すことができます。

## 追加の例

### 式をテストする

一部のデータテストでは複数の列が必要となるため、`columns:` キーの下に列をネストするのは意味がありません。このような場合は、代わりにモデル（またはソース、シード、スナップショット）にデータテストを適用できます:

<File name='models/orders.yml'>

```yaml
version: 2

models:
  - name: orders
    description: 
        Order overview data mart, offering key details for each order including if it's a customer's first order and a food vs. drink item breakdown. One row per order.
    tests:
      - dbt_utils.expression_is_true:
          expression: "order_items_subtotal = subtotal"
      - dbt_utils.expression_is_true:
          expression: "order_total = subtotal + tax_paid"
```
</File>

この例では、`order_items_subtotal` が `subtotal` と等しく、`order_total` が `subtotal` と `tax_paid` を正しく合計することを確認するための式のテストに重点を置いています。

### カスタム汎用テストを使用する

独自のカスタム汎用テストを定義している場合は、それを `test_name` として使用できます:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - primary_key  # name of my custom generic test

```

</File>

詳細については、[カスタム汎用テスト](/best-practices/writing-custom-generic-tests)の作成に関するガイドをご覧ください。

### カスタムデータテスト名

デフォルトでは、dbt は以下の要素を連結して汎用テストの名前を生成します。
- テスト名 (`not_null`、`unique` など)
- モデル名 (またはソース/シード/スナップショット)
- 列名 (該当する場合)
- 引数 (該当する場合、例: `accepted_values` の場合は `values`)

テストの設定は含まれません。連結された名前が長すぎる場合、dbt は代わりに短縮されハッシュ化された名前を使用します。これは、テストを含むプロジェクト内のすべてのリソースの一意の識別子を維持することを目的としています。

`name` プロパティを使用して、特定のテストに独自の名前を定義することもできます。

**どのような場合にこれが必要なのでしょうか？** dbt のデフォルトのアプローチでは、奇妙で見苦しいテスト名が生成されることがあります。カスタム名を定義することで、ログメッセージやメタデータアーティファクトにおけるテストの表示方法を完全に制御できます。その名前でテストを選択することもできます。

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: status
        tests:
          - accepted_values:
              name: unexpected_order_status_today
              values: ['placed', 'shipped', 'completed', 'returned']
              config:
                where: "order_date = current_date"
```

</File>

```sh
$ dbt test --select unexpected_order_status_today
12:43:41  Running with dbt=1.1.0
12:43:41  Found 1 model, 1 test, 0 snapshots, 0 analyses, 167 macros, 0 operations, 1 seed file, 0 sources, 0 exposures, 0 metrics
12:43:41
12:43:41  Concurrency: 5 threads (target='dev')
12:43:41
12:43:41  1 of 1 START test unexpected_order_status_today ................................ [RUN]
12:43:41  1 of 1 PASS unexpected_order_status_today ...................................... [PASS in 0.03s]
12:43:41
12:43:41  Finished running 1 test in 0.13s.
12:43:41
12:43:41  Completed successfully
12:43:41
12:43:41  Done. PASS=1 WARN=0 ERROR=0 SKIP=0 TOTAL=1
```

テスト名は、特定のモデルと列の組み合わせで定義されたすべてのテストで一意である必要があります。複数の異なる列、または複数の異なるモデルで定義されたテストに同じ名前を付けた場合、`dbt test --select <repeated_custom_name>` はそれらすべてを選択します。

**この機能が必要になるのはどのような場合ですか？** 設定のみが異なる同じテストを2回定義した場合、dbt はこれらのテストを重複テストと見なします:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: status
        tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
              config:
                where: "order_date = current_date"
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
              config:
                # only difference is in the 'where' config
                where: "order_date = (current_date - interval '1 day')" # PostgreSQL syntax
```

</File>

```sh
Compilation Error
  dbt found two tests with the name "accepted_values_orders_status__placed__shipped__completed__returned" defined on column "status" in "models.orders".

  Since these resources have the same name, dbt will be unable to find the correct resource
  when running tests.

  To fix this, change the name of one of these resources:
  - test.testy.accepted_values_orders_status__placed__shipped__completed__returned.69dce9e5d5 (models/one_file.yml)
  - test.testy.accepted_values_orders_status__placed__shipped__completed__returned.69dce9e5d5 (models/one_file.yml)
```

カスタム名を指定すると、dbt がテストを区別しやすくなります:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: status
        tests:
          - accepted_values:
              name: unexpected_order_status_today
              values: ['placed', 'shipped', 'completed', 'returned']
              config:
                where: "order_date = current_date"
          - accepted_values:
              name: unexpected_order_status_yesterday
              values: ['placed', 'shipped', 'completed', 'returned']
              config:
                where: "order_date = (current_date - interval '1 day')" # PostgreSQL
```

</File>

```sh
$ dbt test
12:48:03  Running with dbt=1.1.0-b1
12:48:04  Found 1 model, 2 tests, 0 snapshots, 0 analyses, 167 macros, 0 operations, 1 seed file, 0 sources, 0 exposures, 0 metrics
12:48:04
12:48:04  Concurrency: 5 threads (target='dev')
12:48:04
12:48:04  1 of 2 START test unexpected_order_status_today ................................ [RUN]
12:48:04  2 of 2 START test unexpected_order_status_yesterday ............................ [RUN]
12:48:04  1 of 2 PASS unexpected_order_status_today ...................................... [PASS in 0.04s]
12:48:04  2 of 2 PASS unexpected_order_status_yesterday .................................. [PASS in 0.04s]
12:48:04
12:48:04  Finished running 2 tests in 0.21s.
12:48:04
12:48:04  Completed successfully
12:48:04
12:48:04  Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```

**[`store_failures`](/reference/resource-configs/store_failures) を使用する場合:** dbt は、各データテストの名前を、失敗したレコードを保存するテーブルの名前として使用します。あるテストにカスタム名を定義している場合は、そのカスタム名がその失敗テーブルにも使用されます。オプションで、テストの [`alias`](/reference/resource-configs/alias) を設定して、テスト名（メタデータ用）とデータベーステーブル名（失敗の保存用）の両方を個別に制御することもできます。

### テスト定義の代替形式

複数の引数と設定を含む汎用データテストを定義する場合、YAML は見た目も操作性も複雑になることがあります。より簡潔にしたい場合は、テスト名を `test_name` として指定することで、同じテストプロパティを単一の辞書の最上位キーとして定義できます。これは完全にあなた次第です。

この例は上記の例と同じです:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: status
        tests:
          - name: unexpected_order_status_today
            test_name: accepted_values  # name of the generic test to apply
            values:
              - placed
              - shipped
              - completed
              - returned
            config:
              where: "order_date = current_date"
```

</File>
