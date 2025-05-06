---
title: 列タイプを定義するにはどうすればよいですか?
description: "キャスト関数を使用して列の型を定義する"
sidebar_label: '列タイプを定義する方法'
id: define-a-column-type

---

ウェアハウスのSQLエンジンは、ソースまたはモデル内の列に関わらず、すべての列に[データ型](https://www.w3schools.com/sql/sql_datatypes.asp)を自動的に割り当てます。SQLで列を特定のデータ型で処理するように強制するには、`cast`関数を使用します:

<File name='models/order_prices.sql'>

```sql
select
    cast(order_id as integer),
    cast(order_price as double(6,2)) -- a more generic way of doing type conversion
from {{ ref('stg_orders') }}

```

</File>

多くの最新の <Term id="data-warehouse" /> では、`cast( as )` の省略形として `::` 構文がサポートされるようになりました。

<File name='models/orders_prices_colon_syntax.sql'>

```sql
select
    order_id::integer,
    order_price::numeric(6,2) -- you might find this in Redshift, Snowflake, and Postgres
from {{ ref('stg_orders') }}

```

</File>

データの読み込みとキャストは必ずしも期待通りの結果をもたらすとは限らず、ウェアハウスごとに微妙な違いがあります。特定のキャストは許可されない場合があります（例：BigQueryでは、`boolean`型の値を`float64`にキャストすることはできません）。精度の低下を伴うキャスト（例：`float`から`integer`）は、SQLエンジンによる推測や、競合サービスでは使用されていない特定のスキーマへの準拠に依存します。キャストを実行する際は、ウェアハウスのキャストルールを理解し、ソースとモデルのフィールドに最適なラベルを付けることが不可欠です。

ありがたいことに、人気のデータベース サービスには、[Redshift](https://docs.amazonaws.cn/en_us/redshift/latest/dg/r_CAST_function.html) や [Bigquery](https://cloud.google.com/bigquery/docs/reference/standard-sql/conversion_rules) などのタイプのドキュメントが用意されている傾向があります。
