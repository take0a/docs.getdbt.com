---
title: "source の schema プロパティの定義"
sidebar_label: "schema"
resource_types: sources
datatype: schema_name
---

<File name='models/<filename>.yml'>

```yml
version: 2

[sources](/reference/source-properties):
  - name: <source_name>
    database: <database_name>
    schema: <schema_name>
    tables:
      - name: <table_name>
      - ...

```

</File>

## 定義

データベースに保存されているスキーマ名。

このパラメータは、スキーマ名とは異なる [source](/reference/source-properties) 名を使用する場合に便利です。


:::info BigQuery 用語

BigQuery を使用している場合は、_dataset_ 名を `schema` プロパティとして使用します。

:::

## デフォルト

デフォルトでは、dbt はソースの `name` パラメータをスキーマ名として使用します。

## 例
### ソーススキーマには、データベース内のスキーマよりも単純な名前を使用します。

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: jaffle_shop
    schema: postgres_backend_public_schema
    tables:
      - name: orders

```

</File>


ダウンストリームモデルの場合:

```sql
select * from {{ source('jaffle_shop', 'orders') }}
```

次のようにコンパイルされます:

```sql
select * from postgres_backend_public_schema.orders
```
