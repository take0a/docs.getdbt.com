---
title: ソースがターゲット データベースとは異なるデータベースにある場合はどうなりますか?
description: "データベースプロパティを使用して、異なるデータベースのソースを定義します。"
sidebar_label: 'ソースはターゲット データベースとは異なるデータベースにあります'
id: source-in-different-database

---

[`database` プロパティ](/reference/resource-properties/database) を使用して、ソースが存在するデータベースを定義します。

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
