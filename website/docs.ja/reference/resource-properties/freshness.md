<File name='models/<filename>.yml'>

```yaml

version: 2

sources:
  - name: <source_name>
    freshness:
      warn_after:
        [count](#count): <positive_integer>
        [period](#period): minute | hour | day
      error_after:
        [count](#count): <positive_integer>
        [period](#period): minute | hour | day
      [filter](#filter): <boolean_sql_expression>
    [loaded_at_field](#loaded_at_field): <column_name_or_expression>
    [loaded_at_query](#loaded_at_query) <sql_expression> # v1.10 or higher. Should not be used if loaded_at_field is defined

    tables:
      - name: <table_name>
        freshness:
          warn_after:
            [count](#count): <positive_integer>
            [period](#period): minute | hour | day
          error_after:
            [count](#count): <positive_integer>
            [period](#period): minute | hour | day
          [filter](#filter): <boolean_sql_expression>
        [loaded_at_field](#loaded_at_field): <column_name_or_expression>
        [loaded_at_query](#loaded_at_query) <sql_expression> # v1.10 or higher. Should not be used if loaded_at_field is defined

        ...
```

</File>

## 定義
 freshness ブロックは、<Term id="table" /> が「最新」であるとみなされる、最新のレコードから現在までの許容時間を定義するために使用されます。

`freshness` ブロックでは、`warn_after` と `error_after` のどちらか、または両方を指定できます。どちらも指定されていない場合、dbt はこのソース内のテーブルのフレッシュネススナップショットを計算しません。

ほとんどの場合、`loaded_at_field` は必須です。一部のアダプタは、ウェアハウスのメタデータテーブルからソースのフレッシュネスを計算する機能をサポートしており、`loaded_at_field` を除外できます。 <VersionBlock firstVersion="1.10">または、`loaded_at_query` を定義して、カスタム SQL 式を使用してタイムスタンプを計算することもできます。</VersionBlock>

ソースに `freshness:` ブロックがある場合、dbt はその source の鮮度を計算します。
- `loaded_at_field` が指定されている場合、dbt は選択クエリを使用して鮮度を計算します。
- `loaded_at_field` が指定されていない場合、dbt は可能な場合はウェアハウスメタデータテーブルを使用して鮮度を計算します（サポートされているアダプタの v1.7 の新機能）。

<VersionBlock firstVersion="1.10"> 
- `loaded_at_query` が指定されている場合、dbt は指定されたカスタム SQL クエリを使用して鮮度を計算します。
- `loaded_at_query` が指定されている場合、`loaded_at_field` は設定しないでください。
</VersionBlock>


現在、ウェアハウス メタデータ テーブルからの freshness 計算は、以下のアダプタでサポートされています。
- [Snowflake](/reference/resource-configs/snowflake-configs)
- [Redshift](/reference/resource-configs/redshift-configs)
- [BigQuery](/reference/resource-configs/bigquery-configs) ([`dbt-bigquery`](https://github.com/dbt-labs/dbt-bigquery) バージョン 1.7.3 以降でサポート)

[Spark](/reference/resource-configs/spark-configs) アダプタも近日中にサポートされる予定です。

freshness ブロックは階層的に適用されます。
-  source に追加された `freshness` プロパティと `loaded_at_field` プロパティは、その source で定義されているすべてのテーブルに適用されます。
-  source テーブルに追加された `freshness` プロパティと `loaded_at_field` プロパティは、 source に適用されているすべてのプロパティをオーバーライドします。

これは、 source 内のすべてのテーブルが同じ `loaded_at_field` を持つ場合に便利です。よくあるケースです。

 source を freshness計算から除外するには、次の 2 つの方法があります。
- `freshness:` ブロックを追加しない。
- 明示的に `freshness: null` を設定する。

## loaded_at_field

ウェアハウスメタデータテーブルからの鮮度情報の取得をサポートするアダプタではオプション、それ以外の場合は必須です。
<br/><br/>freshness を示すタイムスタンプを返す列名（または式）。

日付フィールドを使用する場合は、タイムスタンプへのキャストが必要になる場合があります。

```yml
loaded_at_field: "completed_date::timestamp"
```

または、SQL バリアントに応じて次のようになります:

```yml
loaded_at_field: "CAST(completed_date AS TIMESTAMP)"
```

UTC 以外のタイムスタンプを使用する場合は、まず UTC にキャストします:

```yml
loaded_at_field: "convert_timezone('Australia/Sydney', 'UTC', created_at_local)"
```

<VersionBlock firstVersion="1.10">

## loaded_at_query

 source の `maxLoadedAt` タイムスタンプを生成するためのカスタム SQL を指定します（ウェアハウスのメタデータや `loaded_at_field` 設定ではなく）。

例:

```yaml

sources:
  - name: your_source
    freshness:
      error_after:
        count: 2
        period: hour
    loaded_at_query: |
      select max(_sdc_batched_at) from (
      select * from {{ this }}
      where _sdc_batched_at > dateadd(day, -7, current_date)
      qualify count(*) over (partition by _sdc_batched_at::date) > 2000
      )

```

```yaml

sources: 
  - name: ecom
    schema: raw
    description: E-commerce data for the Jaffle Shop
    freshness:
      warn_after:
        count: 24
        period: hour
    tables:
      - name: raw_orders
        description: One record per order
        loaded_at_query: "select {{ current_timestamp() }}"
...

```

`loaded_at_field` も設定されている場合、これを設定する必要はありません。ただし、設定されている場合、dbt はテーブルに最も近い値を使用します。

[フィルター](#filter) は `loaded_at_query` では機能しません。

</VersionBlock>

## count
(必須)

データソースがまだ「最新」とみなされる期間の数を表す正の整数。

## period
(必須)

freshness 計算に使用する期間。「分」、「時」、「日」のいずれかです。

## filter
(オプション)

`dbt source freshness` で実行されるクエリに where 句を追加して、スキャンするデータを制限します。

このフィルタは、dbt の source フレッシュネスクエリにのみ適用され、 source テーブルの他の使用には影響しません。

これは特に次の場合に役立ちます。
- BigQuery を使用しており、 source テーブルが [パーティション分割テーブル](https://cloud.google.com/bigquery/docs/partitioned-tables) である場合
- 大規模なテーブルで Snowflake、Databricks、または Spark を使用しており、これによりパフォーマンスが向上する場合


## 例

### 完全な例

<File name='models/<filename>.yml'>

```yaml

version: 2

sources:
  - name: jaffle_shop
    database: raw

    freshness: # default freshness
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}

    loaded_at_field: _etl_loaded_at

    tables:
      - name: customers # this will use the freshness defined above

      - name: orders
        freshness: # make this a little more strict
          warn_after: {count: 6, period: hour}
          error_after: {count: 12, period: hour}
          # Apply a where clause in the freshness query
          filter: datediff('day', _etl_loaded_at, current_timestamp) < 2


      - name: product_skus
        freshness: # do not check freshness for this table
```

</File>

`dbt source freshness` を実行すると、次のクエリが実行されます。

<Tabs
  defaultValue="compiled"
  values={[
    { label: 'Compiled SQL', value: 'compiled', },
    { label: 'Jinja SQL', value: 'jinja', },
  ]
}>
<TabItem value="compiled">

```sql
select
  max(_etl_loaded_at) as max_loaded_at,
  convert_timezone('UTC', current_timestamp()) as snapshotted_at
from raw.jaffle_shop.orders

where datediff('day', _etl_loaded_at, current_timestamp) < 2

```

</TabItem>

<TabItem value="jinja">

```sql
select
  max({{ loaded_at_field }}) as max_loaded_at,
  {{ current_timestamp() }} as snapshotted_at
from {{ source }}
{% if filter %}
where {{ filter }}
{% endif %}
```

_[ソースコード](https://github.com/dbt-labs/dbt-core/blob/HEAD/core/dbt/include/global_project/macros/adapters/common.sql#L262)_
</TabItem>

</Tabs>
