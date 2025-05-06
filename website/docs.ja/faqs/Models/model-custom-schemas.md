---
title: ターゲット スキーマ以外のスキーマでモデルを構築したり、モデルを複数のスキーマに分割したりできますか?
description: "ターゲットスキーマ外でモデルを構築できる"
sidebar_label: 'ターゲットスキーマ外のスキーマでモデルを構築する方法'
id: model-custom-schemas

---

はい！`dbt_project.yml` ファイル内の [schema](reference/resource-configs/schema.md) 構成を使用するか、`config` ブロックを使用します:

<File name='dbt_project.yml'>

```yml

name: jaffle_shop
...

models:
  jaffle_shop:
    marketing:
      schema: marketing # seeds in the `models/mapping/ subdirectory will use the marketing schema
```

</File>

<File name='models/customers.sql'>

```sql
{{
  config(
    schema='core'
  )
}}
```

</File>
