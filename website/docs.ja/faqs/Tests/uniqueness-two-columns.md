---
title: 2 つの列の一意性をテストできますか?
description: "2つの列の一意性をテストするオプション"
sidebar_label: 'Test the uniqueness of two columns'
id: uniqueness-two-columns

---

はい、いくつかの選択肢があります。


複数の国からのレコードが含まれ、ID と国コードの組み合わせが一意である注文 <Term id="table" /> を考えてみましょう:

| order_id | country_code |
|----------|--------------|
| 1        | AU           |
| 2        | AU           |
| ...      | ...          |
| 1        | US           |
| 2        | US           |
| ...      | ...          |


いくつかのアプローチを以下に示します:

#### 1. モデルに一意のキーを作成し、それをテストします

<File name='models/orders.sql'>

```sql

select
  country_code || '-' || order_id as surrogate_key,
  ...

```

</File>

<File name='models/orders.yml'>

```yml
version: 2

models:
  - name: orders
    columns:
      - name: surrogate_key
        tests:
          - unique

```

</File>


#### 2. 式をテストする

<File name='models/orders.yml'>

```yml
version: 2

models:
  - name: orders
    tests:
      - unique:
          column_name: "(country_code || '-' || order_id)"
```

</File>


#### 3. `dbt_utils.unique_combination_of_columns`テストを使用する

これはパフォーマンスが向上するため、特に大規模なデータセットで役立ちます。詳細については、[パッケージ](/docs/build/packages)のドキュメントをご覧ください。

<File name='models/orders.yml'>

```yml
version: 2

models:
  - name: orders
    tests:
      - dbt_utils.unique_combination_of_columns:
          combination_of_columns:
            - country_code
            - order_id
```

</File>
