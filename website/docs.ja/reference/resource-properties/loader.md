---
resource_types: sources
datatype: string
---

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <source_name>
    database: <database_name>
    loader: <string>
    tables:
      - ...

```

</File>

## 定義

このソースをウェアハウスにロードするツールについて記述します。このプロパティはドキュメント作成のみを目的としており、dbt では意味のある用途で使用されません。

## 例

### どのELツールがデータをロードしたかを示す

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: jaffle_shop
    loader: fivetran
    tables:
      - name: orders
      - name: customers

  - name: stripe
    loader: stitch
    tables:
      - name: payments
```

</File>
