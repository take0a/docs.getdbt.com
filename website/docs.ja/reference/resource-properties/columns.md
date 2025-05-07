---
resource_types: all
datatype: test
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
    columns:
      - name: <column_name>
        data_type: <string>
        [description](/reference/resource-properties/description): <markdown_string>
        [quote](/reference/resource-properties/columns#quote): true | false
        [tests](/reference/resource-properties/data-tests): ...
        [tags](/reference/resource-configs/tags): ...
        [meta](/reference/resource-configs/meta): ...
      - name: <another_column>
        ...
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
      columns:
        - name: <column_name>
          [description](/reference/resource-properties/description): <markdown_string>
          data_type: <string>
          [quote](/reference/resource-properties/columns#quote): true | false
          [tests](/reference/resource-properties/data-tests): ...
          [tags](/reference/resource-configs/tags): ...
          [meta](/reference/resource-configs/meta): ...
        - name: <another_column>
          ...

```

</File>

</TabItem>

<TabItem value="seeds">

<File name='seeds/<filename>.yml'>

```yml
version: 2

seeds:
  - name: <seed_name>
    columns:
      - name: <column_name>
        [description](/reference/resource-properties/description): <markdown_string>
        data_type: <string>
        [quote](/reference/resource-properties/columns#quote): true | false
        [tests](/reference/resource-properties/data-tests): ...
        [tags](/reference/resource-configs/tags): ...
        [meta](/reference/resource-configs/meta): ...
      - name: <another_column>
            ...
```

</File>

</TabItem>

<TabItem value="snapshots">

<File name='snapshots/<filename>.yml'>

```yml
version: 2

snapshots:
  - name: <snapshot_name>
    columns:
      - name: <column_name>
        [description](/reference/resource-properties/description): <markdown_string>
        data_type: <string>
        [quote](/reference/resource-properties/columns#quote): true | false
        [tests](/reference/resource-properties/data-tests): ...
        [tags](/reference/resource-configs/tags): ...
        [meta](/reference/resource-configs/meta): ...
      - name: <another_column>

```

</File>

</TabItem>


<TabItem value="analyses">

<File name='analyses/<filename>.yml'>

```yml
version: 2

analyses:
  - name: <analysis_name>
    columns:
      - name: <column_name>
        [description](/reference/resource-properties/description): <markdown_string>
        data_type: <string>
      - name: <another_column>

```

</File>

</TabItem>

</Tabs>

列自体はリソースではありません。別のリソースタイプの子プロパティです。列には、リソースレベルで定義されたプロパティに類似したサブプロパティを定義できます:
- `tags`
- `meta`
- `tests`
- `description`

列はリソースではないため、`tags` プロパティと `meta` プロパティは真の設定ではありません。親リソースの `tags` 値や `meta` 値は継承されません。ただし、列または最上位リソースに適用されたタグを使用して、列に定義された汎用テストを選択することは可能です。[テスト選択の例](/reference/node-selection/test-selection-examples#run-tests-on-tagged-columns) を参照してください。

列ではオプションで `data_type` を定義できます。これは以下の目的で必要です。
- モデル [contract](/reference/resource-configs/contract) の適用
- ソースの [`external`](/reference/resource-properties/external) プロパティや [`dbt-external-tables`](https://hub.getdbt.com/dbt-labs/dbt_external_tables/latest/) など、他のパッケージやプラグインでの使用

### `quote`

`quote` フィールドを使用すると、列名の引用を有効または無効にすることができます。

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

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: model_name
    columns:
      - name: column_name
        quote: true | false

```

</File>

</TabItem>

<TabItem value="sources">

<File name='models/schema.yml'>

```yml
version: 2

sources:
  - name: source_name
    tables:
      - name: table_name
        columns:
          - name: column_name
            quote: true | false

```

</File>

</TabItem>

<TabItem value="seeds">

<File name='seeds/schema.yml'>

```yml
version: 2

seeds:
  - name: seed_name
    columns:
      - name: column_name
        quote: true | false

```

</File>

</TabItem>

<TabItem value="snapshots">

<File name='snapshots/schema.yml'>

```yml
version: 2

snapshots:
  - name: snapshot_name
    columns:
      - name: column_name
        quote: true | false

```

</File>

</TabItem>

<TabItem value="analyses">

<File name='analysis/schema.yml'>

```yml
version: 2

analyses:
  - name: analysis_name
    columns:
      - name: column_name
        quote: true | false

```

</File>

</TabItem>

</Tabs>

### デフォルト

デフォルトの引用符の値は「false」です

### 説明

これは、引用符の扱いが特に不安定なSnowflakeを使用している場合に特に重要です。

このプロパティは、次の場合に役立ちます。
- ソース <Term id="table" /> に、選択時に引用符で囲む必要がある列がある場合（例：列の大文字と小文字の区別を維持する場合）
- Snowflakeでシードが `quote_columns: true` ([docs](/reference/resource-configs/quote_columns)) で作成された場合
- モデルがSQLで引用符を使用している場合（予約語の使用を回避するためなど）

```sql
select user_group as "group"
```

`quote: true` を設定しない場合：
- この列に適用された [データテスト](/docs/build/data-tests) は、無効な SQL が原因で失敗する可能性があります。
- ドキュメントが正しく表示されない可能性があります。例: `group` と `"group"` が同じ列名として一致しない可能性があります。

### 例

#### ソーステーブル内の引用符で囲まれた列にテストを追加する

これは特にSnowflakeを使用する場合に重要です:

```yml
version: 2

sources:
  - name: stripe
    tables:
      - name: payment
        columns:
          - name: orderID
            quote: true
            tests:
              - not_null

```

`quote: true` がない場合、次のエラーが発生します:

```
$ dbt test -s source:stripe.*
Running with dbt=0.16.1
Found 7 models, 22 tests, 0 snapshots, 0 analyses, 130 macros, 0 operations, 0 seed files, 4 sources

13:33:37 | Concurrency: 4 threads (target='learn')
13:33:37 |
13:33:37 | 1 of 1 START test source_not_null_stripe_payment_order_id............ [RUN]
13:33:39 | 1 of 1 ERROR source_not_null_stripe_payment_order_id................. [ERROR in 1.89s]
13:33:39 |
13:33:39 | Finished running 1 tests in 6.43s.

Completed with 1 error and 0 warnings:

Database Error in test source_not_null_stripe_payment_order_id (models/staging/stripe/src_stripe.yml)
  000904 (42000): SQL compilation error: error line 3 at position 6
  invalid identifier 'ORDERID'
  compiled SQL at target/compiled/jaffle_shop/schema_test/source_not_null_stripe_payment_orderID.sql
```

これは、dbt が実行しようとしているためです:

```sql
select count(*)
from raw.stripe.payment
where orderID is null

```

の代わりに：

```sql
select count(*)
from raw.stripe.payment
where "orderID" is null

```
