---
resource_types: [tests]
datatype: boolean
---

設定されたテストは、`dbt test --store-failures` が呼び出されると、失敗を保存します。この設定を `false` に設定し、[`store_failures_as`](/reference/resource-configs/store_failures_as) が設定されている場合は、設定が上書きされます。

## 説明

オプションで、テストの失敗をデータベースに常に保存するか、保存しないかを設定します。
- `true` または `false` を指定した場合、`store_failures` 設定は `--store-failures` フラグの有無よりも優先されます。
- `store_failures` 設定が `none` または省略されている場合、リソースは `--store-failures` フラグの値を使用します。
- true の場合、`store_failures` はテストに失敗したすべてのレコード（[limit](/reference/resource-configs/limit) まで）を保存します。失敗は、テスト名の新しいテーブルに保存されます。
- テストの結果は、同じテストで失敗が発生しなかった場合でも、常に以前の失敗を **置き換え** ます。
- デフォルトでは、`store_failures` は `{{ profile.schema }}_dbt_test__audit` というスキーマを使用しますが、このスキーマを別の値に[構成](/reference/resource-configs/schema#tests)できます。作業に必要なスキーマを作成またはアクセスする権限があることを確認してください。詳細については、[FAQ](#faqs) を参照してください。

このロジックは、[`should_store_failures()`](https://github.com/dbt-labs/dbt-adapters/blob/60005a0a2bd33b61cb65a591bc1604b1b3fd25d5/dbt/include/global_project/macros/materializations/configs.sql#L15) マクロにエンコードされています。


<Tabs
  defaultValue="specific"
  values={[
    { label: 'Specific test', value: 'specific', },
    { label: 'Singular test', value: 'singular', },
    { label: 'Generic test block', value: 'generic', },
    { label: 'Project level', value: 'project', },
  ]
}>

<TabItem value="specific">

汎用 (スキーマ) テストの特定のインスタンスを構成します:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: my_model
    columns:
      - name: my_column
        tests:
          - unique:
              config:
                store_failures: true  # always store failures
          - not_null:
              config:
                store_failures: false  # never store failures
```

</File>

</TabItem>

<TabItem value="singular">

特異（データ）テストを構成します:

<File name='tests/<filename>.sql'>

```sql
{{ config(store_failures = true) }}

select ...
```

</File>

</TabItem>

<TabItem value="generic">

テスト ブロック (定義) 内に構成を設定して、汎用 (スキーマ) テストのすべてのインスタンスのデフォルトを設定します:

<File name='macros/<filename>.sql'>

```sql
{% test <testname>(model, column_name) %}

{{ config(store_failures = false) }}

select ...

{% endtest %}
```

</File>

</TabItem>

<TabItem value="project">

パッケージまたはプロジェクト内のすべてのテストのデフォルトを設定します:

<File name='dbt_project.yml'>

```yaml
tests:
  +store_failures: true  # all tests
  
  <package_name>:
    +store_failures: false # tests in <package_name>
```

</File>

</TabItem>

</Tabs>

## FAQs

<DetailsToggle alt_header=" 'permissions denied for schema' というエラーが表示される">

`Adapter name adapter: Adapter_name error: permission denied for schema dev_username_dbt_test__audit` というエラーが表示される場合、開発スキーマへのオーナーアクセス権は持っているものの、ユーザーに新しいスキーマを作成する権限がないことが原因である可能性があります。

この問題を解決するには、カスタムスキーマの作成またはアクセスに対する適切な権限が必要です。各データプラットフォーム環境で次の SQL コマンドを実行してください。正確な権限クエリはデータプラットフォームによって異なる場合がありますのでご注意ください。

```sql
create schema if not exists dev_username_dbt_test__audit authorization username;
```

_`dev_username` を実際の開発スキーマ名に、`username` を権限を付与する適切なユーザーに置き換えてください。_

このコマンドは、`store_failures` 構成でよく使用される `dbt_test__audit` スキーマの作成とアクセスに必要な権限を付与します。

</DetailsToggle>
