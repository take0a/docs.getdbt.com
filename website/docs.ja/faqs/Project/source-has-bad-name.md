---
title: ソースが不適切な名前のスキーマまたはテーブル内にある場合はどうなりますか?
description: "スキーマと識別子のプロパティを使用して名前を定義する"
sidebar_label: 'ソースが不適切な名前のスキームまたはテーブル内にある'
id: source-has-bad-name

---

デフォルトでは、dbt は `name:` パラメータを使用してソース参照を構築します。

これらの名前が少し不完全な場合は、[schema](/reference/resource-properties/schema) プロパティと [identifier](/reference/resource-properties/identifier) プロパティを使用してデータベースに従って名前を定義し、`name:` プロパティを使用して適切な名前を指定してください。

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: jaffle_shop
    schema: postgres_backend_public_schema
    database: raw
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
select * from raw.postgres_backend_public_schema.api_orders
```
