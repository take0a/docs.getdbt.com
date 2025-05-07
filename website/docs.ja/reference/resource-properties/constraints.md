---
resource_types: [models]
datatype: "{dictionary}"
---

制約は多くのデータプラットフォームに備わっている機能です。制約を指定すると、プラットフォームは新しいテーブルにデータを入力する際、または既存のテーブルにデータを挿入する際に、追加の検証を実行します。検証に失敗した場合、テーブルの作成または更新は失敗し、操作はロールバックされ、明確なエラーメッセージが表示されます。

制約を適用すると、モデルによってマテリアライズされたテーブルに無効なデータが含まれることがなくなります。適用方法はデータプラットフォームによって大きく異なります。

制約を適用するには、モデル [コントラクト](/reference/resource-configs/contract) の宣言と適用が必要です。

**制約は「エフェメラル」モデルや「ビュー」としてマテリアライズされたモデルには適用されません**。制約の適用と強制は「テーブル」モデルと「増分」モデルでのみサポートされます。

## 制約の定義

制約は、単一の列に対して定義することも、モデルレベルで1つ以上の列に対して定義することもできます。原則として、単一列制約はそれらの列に直接定義することをお勧めします。

単一のモデルに対して複数の `primary_key` 制約を定義する場合、それらはモデルレベルで定義する必要があります。列レベルで複数の `primary_key` 制約を定義することはサポートされていません。

制約の構造は次のとおりです。
- `type` (必須): `not_null`、`unique`、`primary_key`、`foreign_key`、`check`、`custom` のいずれか
- `expression`: 制約を修飾するフリーテキスト入力。特定の制約タイプでは必須、その他のタイプではオプションです。
- `name` (オプション): この制約のわかりやすい名前。一部のデータプラットフォームでサポートされています。
- `columns` (モデル レベルのみ): 制約を適用する列名のリスト。

<VersionBlock firstVersion="1.9">

外部キー制約には、以下の 2 つの追加入力があります。
- `to`: 参照先テーブルを示すリレーション入力。[`ref()`](/reference/dbt-jinja-functions/ref)] や [`source()`](/reference/dbt-jinja-functions/source) などが考えられます。
- `to_columns`: 対応する主キーまたは一意キーを含む、そのテーブル内の列のリスト。

外部キーを定義するこの構文では `ref` が使用されているため、依存関係が取得され、さまざまな環境で機能します。[dbt Cloud "最新"](/docs/dbt-versions/cloud-release-tracks) および [dbt Core v1.9+](/docs/dbt-versions/core-upgrade/upgrading-to-v1.9) で使用できます。

<File name='models/schema.yml'>

```yml
models:
  - name: <model_name>
    
    # required
    config:
      contract: {enforced: true}
    
    # model-level constraints
    constraints:
      - type: primary_key
        columns: [first_column, second_column, ...]
      - type: foreign_key # multi_column
        columns: [first_column, second_column, ...]
        to: ref('my_model_to') | source('source', 'source_table')
        to_columns: [other_model_first_column, other_model_second_columns, ...]
      - type: check
        columns: [first_column, second_column, ...]
        expression: "first_column != second_column"
        name: human_friendly_name
      - type: ...
    
    columns:
      - name: first_column
        data_type: string
        
        # column-level constraints
        constraints:
          - type: not_null
          - type: unique
          - type: foreign_key
            to: ref('my_model_to') | source('source', 'source_table')
            to_columns: [other_model_column]
          - type: ...
```

</File>

サポートされている dbt アダプタは、これらのフィールドに値が入力されると、`expression` ではなく外部キー制約をレンダリングします。

外部キー制約をサポートするアダプタの詳細については、[プラットフォーム制約のサポート](/docs/collaborate/govern/model-contracts#platform-constraint-support) に関するガイドをご覧ください。

</VersionBlock>

<VersionBlock lastVersion="1.8">

When using `foreign_key`, you need to specify the referenced table's schema manually. Use `{{ target.schema }}` in the `expression` field to automatically pass the schema used by the target environment:

`expression: "{{ target.schema }}.customers(customer_id)"` 

Note that later versions of dbt will have more efficient ways of handling this. Find out more about upgrading to the latest version, refer to [About dbt Core versions](/docs/dbt-versions/core) or [Upgrade dbt version in Cloud](/docs/dbt-versions/upgrade-dbt-version-in-cloud).

<File name='models/schema.yml'>

```yml
models:
  - name: <model_name>
    
    # required
    config:
      contract: {enforced: true}
    
    # model-level constraints
    constraints:
      - type: primary_key
        columns: [first_column, second_column, ...]
      - type: foreign_key # multi_column
        columns: [first_column, second_column, ...]
        expression: "{{ target.schema }}.other_model_name (other_model_first_column, other_model_second_column, ...)"
      - type: check
        columns: [first_column, second_column, ...]
        expression: "first_column != second_column"
        name: human_friendly_name
      - type: ...
    
    columns:
      - name: first_column
        data_type: string
        
        # column-level constraints
        constraints:
          - type: not_null
          - type: unique
          - type: foreign_key
            expression: "{{ target.schema }}.other_model_name (other_model_column)"
          - type: ...
```

</File>

</VersionBlock>

## プラットフォーム固有のサポート

トランザクションデータベースでは、特定の列の許容値に対して、その値のデータ型だけでなく、より厳密な「制約」を定義することができます。例えば、PostgresはANSI SQL標準のすべての制約（「not null」、「unique」、「primary key」、「foreign key」）をサポートし、適用します。さらに、ブール式を評価する柔軟な行レベルの「check」制約もサポートしています。

ほとんどの分析データプラットフォームは「not null」制約をサポートし、適用しますが、残りの制約はサポートしていないか、適用していません。従来のデータカタログやERDツールとの統合を目的として、「情報」制約（適用されないことを前提としている）を追加することが望ましい場合もあります（[dbt-core#3295](https://github.com/dbt-labs/dbt-core/issues/3295)）。一部のデータプラットフォームでは、追加のキーワードを指定することで、クエリの最適化に主キー制約または外部キー制約をオプションで使用できます。

そのため、フィルターには以下の2つのオプションフィールドを指定できます。
- `warn_unenforced: False` を指定すると、このデータプラットフォームでサポートされているものの強制されていない制約に関する警告がスキップされます。この制約はテンプレートDDLに含まれます。
- `warn_unsupported: False` を指定すると、このデータプラットフォームでサポートされていない制約に関する警告がスキップされ、テンプレートDDLには含まれません。

<WHCode>

<div warehouse="Postgres">

* PostgreSQL 制約のドキュメント: [こちら](https://www.postgresql.org/docs/current/ddl-constraints.html#id-1.5.4.6.6)

<File name='models/constraints_example.sql'>

```sql
{{
  config(
    materialized = "table"
  )
}}

select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
```

</File>

<File name='models/schema.yml'>

```yml
models:
  - name: dim_customers
    config:
      contract:
        enforced: true
    columns:
      - name: id
        data_type: int
        constraints:
          - type: not_null
          - type: primary_key
          - type: check
            expression: "id > 0"
      - name: customer_name
        data_type: text
      - name: first_transaction_date
        data_type: date
```

</File>

制約を強制する予期される DDL:

<File name='target/run/.../constraints_example.sql'>

```sql
create table "database_name"."schema_name"."constraints_example__dbt_tmp"
( 
    id integer not null primary key check (id > 0),
    customer_name text,
    first_transaction_date date    
)
;
insert into "database_name"."schema_name"."constraints_example__dbt_tmp" 
(   
    id,
    customer_name,  
    first_transaction_date
) 
(
select 
    1 as id, 
    'My Favorite Customer' as customer_name, 
    cast('2019-01-01' as date) as first_transaction_date
);
```

</File>

</div>

<div warehouse="Redshift">

Redshiftは現在、「not null」制約のみを適用します。その他の制約はメタデータのみです。また、Redshiftではテーブル作成時に列チェックは許可されません。詳しくは、Redshiftのドキュメント[こちら](https://docs.aws.amazon.com/redshift/latest/dg/t_Defining_constraints.html)をご覧ください。

<File name='models/constraints_example.sql'>

```sql
{{
  config(
    materialized = "table"
  )
}}

select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
```

</File>

<File name='models/schema.yml'>

```yml
models:
  - name: dim_customers
    config:
      contract:
        enforced: true
    columns:
      - name: id
        data_type: integer
        constraints:
          - type: not_null
          - type: primary_key # not enforced  -- will warn & include
          - type: check       # not supported -- will warn & skip
            expression: "id > 0"
        tests:
          - unique            # primary_key constraint is not enforced
      - name: customer_name
        data_type: varchar
      - name: first_transaction_date
        data_type: date
```

Redshiftでは、`varchar`値の最大長がデフォルトで256文字に制限されていることに注意してください（長さを指定しない場合も同様です）。つまり、256文字を超える文字列データは切り捨てられるか、「文字型に対して値が長すぎます」というエラーが返される可能性があります。最大長を許可するには、`varchar(max)`を使用してください。例：`data_type: varchar(max)`

</File>

制約を強制する予期される DDL:

<File name='target/run/.../constraints_example.sql'>

```sql

create table "database_name"."schema_name"."constraints_example__dbt_tmp"
    
(
    id integer not null,
    customer_name varchar,
    first_transaction_date date,
    primary key(id)
)    
;

insert into "database_name"."schema_name"."constraints_example__dbt_tmp"
(   
select
    1 as id,
    'My Favorite Customer' as customer_name,
    cast('2019-01-01' as date) as first_transaction_date
); 
```

</File>


</div>

<div warehouse="Snowflake">

- Snowflakeの制約に関するドキュメント: [こちら](https://docs.snowflake.com/en/sql-reference/constraints-overview.html)
- Snowflakeのデータ型: [こちら](https://docs.snowflake.com/en/sql-reference/intro-summary-data-types.html)

Snowflakeは、`unique`、`not null`、`primary key`、`foreign key`の4種類の制約をサポートしています。

現時点では、`not null`（および`primary key`の`not null`プロパティ）のみが実際にチェックされることに注意してください。
その他の制約は純粋にメタデータであり、データの挿入時には検証されません。 Snowflake は `unique`、`primary`、`foreign_key` 制約を検証しませんが、オプションで制約 `expression` フィールドに [`rely`](https://docs.snowflake.com/en/user-guide/join-elimination) を指定することにより、クエリの最適化にこれらの制約を使用するように Snowflake に指示できます。

現在、Snowflake は `check` 構文をサポートしておらず、dbt プロジェクト内の一部のモデルで `check` 構成が設定されている場合、dbt はそれをスキップして警告メッセージを表示します。

<File name='models/constraints_example.sql'>

```sql
{{
  config(
    materialized = "table"
  )
}}

select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
```

</File>

<File name='models/schema.yml'>

```yml
models:
  - name: dim_customers
    config:
      contract:
        enforced: true
    columns:
      - name: id
        data_type: integer
        description: hello
        constraints:
          - type: not_null
          - type: primary_key # not enforced  -- will warn & include
          - type: check       # not supported -- will warn & skip
            expression: "id > 0"
        tests:
          - unique            # need this test because primary_key constraint is not enforced
      - name: customer_name
        data_type: text
      - name: first_transaction_date
        data_type: date
```

</File>

制約を強制する予期される DDL:

<File name='target/run/.../constraints_example.sql'>

```sql
create or replace transient table <database>.<schema>.constraints_model        
(
    id integer not null primary key,
    customer_name text,
    first_transaction_date date  
)
as
(
select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
);
```

</File>

</div>

<div warehouse="BigQuery">

BigQuery では、`not null` 制約の定義と適用、およびクエリの最適化に使用できる `primary key` 制約と `foreign key` 制約の定義（ただし適用は _not_）が可能です。BigQuery は、その他の制約の定義または適用をサポートしていません。詳細については、[プラットフォーム制約のサポート](/docs/collaborate/govern/model-contracts#platform-constraint-support) をご覧ください。

ドキュメント: https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language

データ型: https://cloud.google.com/bigquery/docs/reference/standard-sql/data-types

<File name='models/constraints_example.sql'>

```sql
{{
  config(
    materialized = "table"
  )
}}

select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
```

</File>

<File name='models/schema.yml'>

```yml
models:
  - name: dim_customers
    config:
      contract:
        enforced: true
    columns:
      - name: id
        data_type: int
        constraints:
          - type: not_null
          - type: primary_key # not enforced  -- will warn & include
          - type: check       # not supported -- will warn & skip
            expression: "id > 0"
        tests:
          - unique            # primary_key constraint is not enforced
      - name: customer_name
        data_type: string
      - name: first_transaction_date
        data_type: date
```

</File>

### ネストされた列に対する列レベルの制約:

<File name='models/nested_column_constraints_example.sql'>

```sql
{{
  config(
    materialized = "table"
  )
}}

select
  'string' as a,
  struct(
    1 as id,
    'name' as name,
    struct(2 as id, struct('test' as again, '2' as even_more) as another) as double_nested
  ) as b
```

</File>

<File name='models/nested_fields.yml'>

```yml
version: 2

models:
  - name: nested_column_constraints_example
    config:
      contract: 
        enforced: true
    columns:
      - name: a
        data_type: string
      - name: b.id
        data_type: integer
        constraints:
          - type: not_null
      - name: b.name
        description: test description
        data_type: string
      - name: b.double_nested.id
        data_type: integer
      - name: b.double_nested.another.again
        data_type: string
      - name: b.double_nested.another.even_more
        data_type: integer
        constraints: 
          - type: not_null
```

</File>

### 制約を強制する予期される DDL:

<File name='target/run/.../constraints_example.sql'>

```sql
create or replace table `<project>`.`<dataset>`.`constraints_model`        
(
    id integer not null,
    customer_name string,
    first_transaction_date date  
)
as
(
select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
);
```

</File>

</div>

<div warehouse="Databricks">

Databricks では、以下の制約を定義できます。

- `not null` 制約
- 条件式に 1 つ以上の列を含む、追加の `check` 制約

Databricks はトランザクションをサポートしておらず、列スキーマを使用した `create or replace table` の使用も許可していないため、まずスキーマなしでテーブルを作成し、その後 `alter` ステートメントを実行してさまざまな制約を追加します。

つまり、以下のようになります。

- 列の名前と順序はチェックされますが、型はチェックされません。
- `constraints` または `constraint_check` が失敗した場合、失敗したデータを含むテーブルはウェアハウス内に残ります。

Databricks における制約のサポートの詳細については、[このページ](https://docs.databricks.com/tables/constraints.html) を参照してください。

<File name='models/constraints_example.sql'>

```sql
{{
  config(
    materialized = "table"
  )
}}

select 
  1 as id, 
  'My Favorite Customer' as customer_name, 
  cast('2019-01-01' as date) as first_transaction_date
```

</File>

<File name='models/schema.yml'>

```yml
models:
  - name: dim_customers
    config:
      contract:
        enforced: true
    columns:
      - name: id
        data_type: int
        constraints:
          - type: not_null
          - type: primary_key # not enforced  -- will warn & include
          - type: check       # not supported -- will warn & skip
            expression: "id > 0"
        tests:
          - unique            # primary_key constraint is not enforced
      - name: customer_name
        data_type: text
      - name: first_transaction_date
        data_type: date
```

</File>

制約を強制する予期される DDL:

<File name='target/run/.../constraints_example.sql'>

```sql
  create or replace table schema_name.my_model 
  using delta 
  as
    select
      1 as id,
      'My Favorite Customer' as customer_name,
      cast('2019-01-01' as date) as first_transaction_date
```

</File>

以下の文が続く

```sql
alter table schema_name.my_model change column id set not null;
alter table schema_name.my_model add constraint 472394792387497234 check (id > 0);
```

</div>

</WHCode>

## カスタム制約

dbt Cloud および dbt Core では、モデルにカスタム制約を適用することで、テーブルの詳細な設定を行うことができます。データウェアハウスによってサポートされる構文と機能は異なります。

カスタム制約を使用すると、特定の列に設定を追加できます。例:

- Create Table As Select (CTAS) を使用する場合は、Snowflake で [マスキングポリシー](https://docs.snowflake.com/en/user-guide/security-column-intro#what-are-masking-policies) を設定します。

- 他のデータ ウェアハウス ([Databricks](https://docs.databricks.com/en/sql/language-manual/sql-ref-syntax-ddl-create-table-using.html) や [BigQuery](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#column_name_and_column_schema) など) には、CTAS ステートメントの列に設定できる独自のパラメータ セットがあります。

制約は、いくつかの方法で実装できます:

<Expandable alt_header="タグでカスタム制約">

次の構文を使用して、契約と制約を含むタグベースのマスキング ポリシーを実装する方法の例を次に示します:

<File name='models/constraints_example.yml'>

```yaml

models:
  - name: my_model
    config:
      contract:
        enforced: true
      materialized: table
    columns:
      - name: id
        data_type: int
        constraints:
          - type: custom
            expression: "tag (my_tag = 'my_value')" #  A custom SQL expression used to enforce a specific constraint on a column.

```

</File>

この構文を使用するには、すべての列とその型を設定する必要があります。これは、`<cols_info_with_masking> mytable as ...` という create または replace を送信する唯一の方法だからです。列のリストの一部だけを指定しても、この構文は実行できません。つまり、列と制約フィールドが完全に定義されている必要があります。

すべての列を含む YAML を生成するには、[dbt-codegen](https://github.com/dbt-labs/dbt-codegen/tree/0.12.1/?tab=readme-ov-file#generate_model_yaml-source) の `generate_model_yaml` を使用できます。

</Expandable>

<Expandable alt_header="タグなしのカスタム制約">

あるいは、タグなしでマスキング ポリシーを追加することもできます:

<File name='models/constraints_example.yml'>
 
```yaml

models:
  - name: my_model
    config:
      contract:
        enforced: true
      materialized: table
    columns:
      - name: id
        data_type: int
        constraints:
          - type: custom
            expression: "masking policy my_policy"

```

</File>
</Expandable>

