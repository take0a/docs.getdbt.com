---
resource_types: sources
datatype: table_identifier
---

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <source_name>
    database: <database_name>
    tables:
      - name: <table_name>
        identifier: <table_identifier>

```

</File>

## 定義

データベースに保存されている <Term id="table" /> の名前。

このパラメータは、データベース内のテーブル名とは異なるソーステーブル名を使用する場合に便利です。

## デフォルト

デフォルトでは、dbt はテーブルの `name` パラメータを識別子として使用します。

## 例

### ソーステーブルには、データベース内のテーブルよりも簡単な名前を使用します。

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: jaffle_shop
    tables:
      - name: orders
        identifier: api_orders

```

</File>


ダウンストリームモデルの場合:

```sql
select * from {{ source('jaffle_shop', 'orders') }}
```

次のようにコンパイルされます:

```sql
select * from jaffle_shop.api_orders
```

### BigQuery でシャード テーブルをソースとして参照する

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: ga
    tables:
      - name: events
        identifier: "events_*"

```

</File>


ダウンストリームモデルの場合:

```sql
select * from {{ source('ga', 'events') }}

-- filter on shards by suffix
where _table_suffix > '20200101'
```

次のようにコンパイルされます:

```sql
select * from `my_project`.`ga`.`events_*`

-- filter on shards by suffix
where _table_suffix > '20200101'
```
