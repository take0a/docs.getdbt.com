---
title: "source の database プロパティの定義"
sidebar_label: "database"
resource_types: sources
datatype: database_name
---

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <source_name>
    database: <database_name>
    tables:
      - name: <table_name>
      - ...

```

</File>

## 定義

ソースが保存されているデータベース。

このパラメータを使用するには、ウェアハウスでデータベース間クエリが許可されている必要があります。

#### BigQuery の用語

BigQuery を使用している場合は、`database:` プロパティとして _project_ 名を使用します。

## デフォルト

デフォルトでは、dbt はターゲットデータベース（つまり、テーブルと <Term id="view">ビュー</Term> を作成しているデータベース）を検索します。

## 例

### `raw`データベースに保存されるソースを定義する

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: jaffle_shop
    database: raw
    tables:
      - name: orders
      - name: customers

```

</File>
