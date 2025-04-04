---
title: "カスタム汎用データテストの作成"
id: "writing-custom-generic-tests"
description: Learn how to define your own custom generic data tests.
displayText: Writing custom generic data tests
hoverSnippet: Learn how to write your own custom generic data tests.
---

dbt には、[Not Null](/reference/resource-properties/data-tests#not-null)、[Unique](/reference/resource-properties/data-tests#unique)、[Relationships](/reference/resource-properties/data-tests#relationships)、[Accepted Values](/reference/resource-properties/data-tests#accepted-values) の汎用データ テストが付属しています。(これらは以前は「スキーマ テスト」と呼ばれていましたが、一部の場所ではまだその名前が使われています。) 内部的には、これらの汎用データ テストは `test` ブロック (マクロなど) として定義されています。

:::info
[dbt-utils](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) や [dbt-expectations](https://hub.getdbt.com/calogica/dbt_expectations/latest/) などのオープンソース パッケージには、多数の汎用データ テストが定義されています。探しているテストがすでにここにある可能性があります。
:::

### 標準引数を使用した汎用テスト

汎用テストは SQL ファイルで定義されます。これらのファイルは、次の 2 つの場所に配置できます。
- `tests/generic/`: つまり、[テスト パス](/reference/project-configs/test-paths) 内の `generic` という名前の特別なサブフォルダー (デフォルトでは `tests/`)
- `macros/`: 理由: 汎用テストはマクロとよく似た動作をするため、これまではこれが汎用テストを定義できる唯一の場所でした。汎用テストが複雑なマクロ ロジックに依存している場合は、マクロと汎用テストを同じファイルで定義する方が便利な場合があります。

独自の汎用テストを定義するには、`<test_name>` という `test` ブロックを作成するだけです。すべての汎用テストは、標準引数の 1 つまたは両方を受け入れる必要があります。
- `model`: テストが定義されているリソース。リレーション名にテンプレート化されます。(リソースがソース、シード、またはスナップショットの場合でも、引数の名前は常に `model` になることに注意してください。)
- `column_name`: テストが定義されている列。すべての汎用テストが列レベルで動作するわけではありませんが、動作する場合は、引数として `column_name` を受け入れる必要があります。

両方の引数を使用する `is_even` スキーマ テストの例を次に示します:

<File name='tests/generic/test_is_even.sql'>

```sql
{% test is_even(model, column_name) %}

with validation as (

    select
        {{ column_name }} as even_field

    from {{ model }}

),

validation_errors as (

    select
        even_field

    from validation
    -- if this is true, then even_field is actually odd!
    where (even_field % 2) = 1

)

select *
from validation_errors

{% endtest %}
```

</File>

この `select` ステートメントが 0 個のレコードを返す場合、指定された `model` 引数内のすべてのレコードは偶数です。代わりに 0 個以外のレコードが返された場合は、`model` 内の少なくとも 1 つのレコードが奇数であり、テストは失敗しています。

この汎用テストを使用するには、モデル、ソース、スナップショット、またはシードの `tests` プロパティで名前で指定します:

<VersionBlock firstVersion="1.9">
<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: users
    columns:
      - name: favorite_number
        tests:
      	  - is_even
            [description](/reference/resource-properties/description): "This is a test"
```

</File>

</VersionBlock>

<VersionBlock lastVersion="1.8">
<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: users
    columns:
      - name: favorite_number
        tests:
      	  - is_even
```

</File>

</VersionBlock>

たった 1 行のコードで、テストが作成されました。この例では、`users` が `is_even` テストに `model` 引数として渡され、`favorite_number` が `column_name` 引数として渡されます。他の列、他のモデルにも同じ行を追加できます。それぞれが、_同じ汎用テスト定義を使用して_、プロジェクトに新しいテストを追加します。

### 汎用データテストロジックに説明を追加する

`macros:` セクションの下に `description` キーを含めることで、データ テストのコア ロジックを提供する Jinja マクロに説明を追加できます。マクロ引数の説明など、マクロに直接説明を追加できます。

次に例を示します:

<File name="macros/generic/schema.yml">
    
```yaml
macros:
  - name: test_not_empty_string
    description: Complementary test to default `not_null` test as it checks that there is not an empty string. It only accepts columns of type string.
    arguments:
      - name: model 
        type: string
        description: Model Name
      - name: column_name
        type: string
        description: Column name that should not be an empty string
```
</File>

この例では、次のようになります:
- `schema.yml` ファイルでカスタム テスト マクロを文書化する場合、マクロ名に `test_` プレフィックスを追加します。たとえば、テスト ブロックの名前が `not_empty_string` の場合、マクロの名前は `test_not_empty_string` になります。
- マクロ レベルで説明を提供し、テストの実行内容と関連するメモを説明しています。
- 各引数 (`model`、`column_name` など) にも、その目的を明確にする説明が含まれています。

### 追加の引数を持つ汎用テスト

`is_even` テストは、追加の引数を指定する必要がなく動作します。`relationships` などの他のテストでは、`model` と `column_name` 以外にも引数が必要です。カスタム テストで標準引数以外の引数が必要な場合は、`field` と `to` が以下に含まれているように、それらの引数をテスト署名に含めます:

<File name='tests/generic/test_relationships.sql'>

```sql
{% test relationships(model, column_name, field, to) %}

with parent as (

    select
        {{ field }} as id

    from {{ to }}

),

child as (

    select
        {{ column_name }} as id

    from {{ model }}

)

select *
from child
where id is not null
  and id not in (select id from parent)

{% endtest %}
```

</File>

`.yml` ファイルからこのテストを呼び出す場合は、テストの引数を辞書で指定します。標準引数 (`model` および `column_name`) はコンテキストによって提供されるため、再度定義する必要はありません。

<VersionBlock lastVersion="1.8">

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: people
    columns:
      - name: account_id
        tests:
          - relationships:
              to: ref('accounts')
              field: id
```

</File>

</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: people
    columns:
      - name: account_id
        tests:
          - relationships:
            [description](/reference/resource-properties/description): "This is a test"
              to: ref('accounts')
              field: id
```

</File>

</VersionBlock>

### デフォルトの設定値を使用した汎用テスト

汎用テスト定義に `config()` ブロックを含めることができます。そこに設定された値は、特定のインスタンスの `.yml` プロパティ内で上書きされない限り、その汎用テストのすべての特定のインスタンスのデフォルトとして設定されます。

<File name='tests/generic/warn_if_odd.sql'>

```sql
{% test warn_if_odd(model, column_name) %}

    {{ config(severity = 'warn') }}

    select *
    from {{ model }}
    where ({{ column_name }} % 2) = 1

{% endtest %}
```

`warn_if_odd` テストが使用されるときはいつでも、特定のテストがその値を上書きしない限り、常に警告レベルの重大度を持ちます:

</File>

<VersionBlock lastVersion="1.8">

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: users
    columns:
      - name: favorite_number
        tests:
      	  - warn_if_odd         # default 'warn'
      - name: other_number
        tests:
          - warn_if_odd:
              severity: error   # overrides
```

</File>

</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: users
    columns:
      - name: favorite_number
        description: "Test favorite_number"
        tests:
      	  - warn_if_odd         # default 'warn'
      - name: other_number
        description: "Test other_number"
        tests:
          - warn_if_odd:
              severity: error   # overrides
```

</File>

</VersionBlock>

### dbt の組み込みテストのカスタマイズ

組み込みの汎用テストの動作方法を変更するには (パラメータの追加、SQL の書き換え、その他の理由など)、`<test_name>` という名前のテスト ブロックを自分のプロジェクトに追加するだけです。dbt はグローバル実装よりも自分のバージョンを優先します。

<File name='tests/generic/<filename>.sql'>

```sql
{% test unique(model, column_name) %}

    -- whatever SQL you'd like!

{% endtest %}
```

</File>

### 例

コミュニティからのカスタム汎用 (「スキーマ」) テストの追加例をいくつか示します:
* [エラーしきい値付きのカスタム スキーマ テストの作成](https://discourse.getdbt.com/t/creating-an-error-threshold-for-schema-tests/966)
* [カスタム スキーマ テストを使用して本番環境でのみテストを実行する](https://discourse.getdbt.com/t/conditionally-running-dbt-tests-only-running-dbt-tests-in-production/322)
* [カスタム スキーマ テストの追加例](https://discourse.getdbt.com/t/examples-of-custom-schema-tests/181)
