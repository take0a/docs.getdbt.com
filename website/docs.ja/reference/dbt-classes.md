---
title: "dbt Classes"
---

dbt には、<Term id="data-warehouse" /> 内のオブジェクト、dbt プロジェクトの一部、およびコマンドの結果を表すために使用するクラスが多数あります。

これらのクラスは、高度な dbt モデルやマクロを構築するときに役立ちます。

## Relation

`Relation` オブジェクトは、適切な引用符を用いてスキーマ名と <Term id="table" /> 名を SQL コードに挿入するために使用されます。`{{ schema }}.{{ table }}` を直接使用して値を挿入するのではなく、常にこのオブジェクトを使用する必要があります。Relation オブジェクトの引用符の指定は、[`quoting` 設定](/reference/project-configs/quoting) で設定できます。

### リレーションの作成

`Relation` は、`Relation` クラスの `create` クラスメソッドを呼び出すことで作成できます。

<File name='Relation.create'>

```python
class Relation:
  def create(database=None, schema=None, identifier=None,
             type=None):
  """
    database (optional): The name of the database for this relation
    schema (optional): The name of the schema (or dataset, if on BigQuery) for this relation
    identifier (optional): The name of the identifier for this relation
    type (optional): Metadata about this relation, eg: "table", "view", "cte"
  """
```

</File>

### リレーションの使用

`api.Relation.create` に加えて、dbt は [`ref`](/reference/dbt-jinja-functions/ref)、[`source`](/reference/dbt-jinja-functions/source)、または [`this`](/reference/dbt-jinja-functions/this) を使用した場合にもリレーションを返します。

<File name='relation_usage.sql'>

```jinja2
{% set relation = api.Relation.create(schema='snowplow', identifier='events') %}

-- Return the `database` for this relation
{{ relation.database }}

-- Return the `schema` (or dataset) for this relation
{{ relation.schema }}

-- Return the `identifier` for this relation
{{ relation.identifier }}

-- Return relation name without the database
{{ relation.include(database=false) }}

-- Return true if the relation is a table
{{ relation.is_table }}

-- Return true if the relation is a view
{{ relation.is_view }}

-- Return true if the relation is a cte
{{ relation.is_cte }}

```

</File>


## Column

`Column` オブジェクトは、リレーション内の列に関する情報をエンコードするために使用されます。

<File name='column.py'>

```python
class Column(object):
  def __init__(self, column, dtype, char_size=None, numeric_size=None):
    """
      column: The name of the column represented by this object
      dtype: The data type of the column (database-specific)
      char_size: If dtype is a variable width character type, the size of the column, or else None
      numeric_size: If dtype is a fixed precision numeric type, the size of the column, or else None
   """


# Example usage:
col = Column('name', 'varchar', 255)
col.is_string() # True
col.is_numeric() # False
col.is_number() # False
col.is_integer() # False
col.is_float() # False
col.string_type() # character varying(255)
col.numeric_type('numeric', 12, 4) # numeric(12,4)
```

</File>

### Column API

### プロパティ

- **char_size**: 文字可変列の最大サイズを返します。
- **column**: 列名を返します。
- **data_type**: 列のデータ型を返します（サイズ/精度/スケールを含む）。
- **dtype**: 列のデータ型を返します（サイズ/精度/スケールを含まない）。
- **name**: 列名を返します（エイリアスとして指定された `column` と同一）。
- **numeric_precision**: 固定小数点列の最大精度を返します。
- **numeric_scale**: 固定小数点列の最大スケールを返します。
- **quoted**: 列名を引用符で囲んで返します。

### インスタンスメソッド

- **is_string()**: 列が文字列型（例：text、varchar）の場合、True を返します。それ以外の場合は False を返します。
- **is_numeric()**: 列が固定精度の数値型（例：`numeric`）の場合、True を返します。それ以外の場合は False を返します。
- **is_number()**: 列が数値型（例：`numeric`、`int`、`float` など）の場合、True を返します。それ以外の場合は False を返します。
- **is_integer()**: 列が整数型（例：`int`、`bigint`、`serial` など）の場合、True を返します。それ以外の場合は False を返します。
- **is_float()**: 列が浮動小数点型（例：`float`、`float64` など）の場合、True を返します。それ以外の場合は False を返します。
- **string_size()**: 列が文字列型の場合、列の幅を返します。それ以外の場合は、例外が発生する

### 静的メソッド

- **string_type(size)**: 文字列型のデータベースで使用可能な表現を返します (例: `character varying(255)`)
- **numeric_type(dtype, precision, scale)**: 数値型のデータベースで使用可能な表現を返します (例: `numeric(12, 4)`)

### 列の使用

<File name='column_usage.sql'>

```jinja2
-- String column
{%- set string_column = api.Column('name', 'varchar', char_size=255) %}

-- Return true if the column is a string
{{ string_column.is_string() }}

-- Return true if the column is a numeric
{{ string_column.is_numeric() }}

-- Return true if the column is a number
{{ string_column.is_number() }}

-- Return true if the column is an integer
{{ string_column.is_integer() }}

-- Return true if the column is a float
{{ string_column.is_float() }}

-- Numeric column
{%- set numeric_column = api.Column('distance_traveled', 'numeric', numeric_precision=12, numeric_scale=4) %}

-- Return true if the column is a string
{{ numeric_column.is_string() }}

-- Return true if the column is a numeric
{{ numeric_column.is_numeric() }}

-- Return true if the column is a number
{{ numeric_column.is_number() }}

-- Return true if the column is an integer
{{ numeric_column.is_integer() }}

-- Return true if the column is a float
{{ numeric_column.is_float() }}

-- Static methods

-- Return the string data type for this database adapter with a given size
{{ api.Column.string_type(255) }}

-- Return the numeric data type for this database adapter with a given precision and scale
{{ api.Column.numeric_type('numeric', 12, 4) }}
```

</File>

## BigQuery columns

BigQuery dbt プロジェクトでは、`Column` 型は `BigQueryColumn` としてオーバーライドされます。このオブジェクトは、追加のプロパティとメソッドを除いて、上記の `Column` 型と同じように動作します。

### プロパティ

- **fields**: フィールドに含まれるサブフィールドのリストを返します（列が構造体の場合）。
- **mode**: 列の「モード」を返します（例: `REPEATED`）。

### インスタンスメソッド

**flatten()**: サブフィールドがそれぞれの列に展開された、フラット化された `BigQueryColumns` リストを返します。例えば、次のネストされたフィールドは、

```
[{"hits": {"pageviews": 1, "bounces": 0}}]
```

will be expanded to:
```
[{"hits.pageviews": 1, "hits.bounces": 0}]
```

## 結果オブジェクト

dbt でリソースを実行すると、`Result` オブジェクトが生成されます。このオブジェクトには、実行されたノード、タイミング、ステータス、アダプタから返されたメタデータに関する情報が含まれます。呼び出しの終了時に、dbt はこれらのオブジェクトを [`run_results.json`](/reference/artifacts/run-results-json) に記録します。

- `node`: 実行された dbt リソース (モデル、シード、スナップショット、テスト) の完全なオブジェクト表現。`unique_id` も含まれます。
- `status`: dbt による実行時の成功、失敗、またはエラーの解釈。
- `thread_id`: このノードを実行したスレッド。例: `Thread-1`
- `execution_time`: このノードの実行にかかった合計時間 (秒単位)。
- `timing`: 実行時間をステップ（多くの場合 `compile` + `execute`）に分割した配列
- `message`: データベースから返された情報に基づいて、dbt が CLI にこの結果をどのように報告するか

import RowsAffected from '/snippets.ja/_run-result.md'; 

<RowsAffected/>
