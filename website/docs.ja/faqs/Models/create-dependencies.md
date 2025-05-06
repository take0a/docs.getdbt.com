---
title: モデル間の依存関係を作成するにはどうすればよいですか?
description: "ref関数を使用して依存関係を作成する"
sidebar_label: 'モデル間の依存関係の作成'
id: create-dependencies

---

`ref` [関数](/reference/dbt-jinja-functions/ref) を使用すると、dbt はモデル間の依存関係を自動的に推測します。

例えば、次のような `customer_orders` モデルを考えてみましょう:

<File name='models/customer_orders.sql'>

```sql
select
    customer_id,
    min(order_date) as first_order_date,
    max(order_date) as most_recent_order_date,
    count(order_id) as number_of_orders
from {{ ref('stg_orders') }}
group by 1

```

</File>

**これらの依存関係を明示的に定義する必要はありません。** dbt は、`stg_orders` モデルを上記のモデル (`customer_orders`) の前にビルドする必要があることを理解します。`dbt run` を実行すると、これらのモデルが順番にビルドされているのが確認できます:

```txt
$ dbt run
Running with dbt=1.6.0
Found 2 models, 28 tests, 0 snapshots, 0 analyses, 130 macros, 0 operations, 0 seed files, 3 sources

11:42:52 | Concurrency: 8 threads (target='dev_snowflake')
11:42:52 |
11:42:52 | 1 of 2 START view model dbt_claire.stg_jaffle_shop__orders........... [RUN]
11:42:55 | 1 of 2 OK creating view model dbt_claire.stg_jaffle_shop__orders..... [CREATE VIEW in 2.50s]
11:42:55 | 2 of 2 START relation dbt_claire.customer_orders..................... [RUN]
11:42:56 | 2 of 2 OK creating view model dbt_claire.customer_orders............. [CREATE VIEW in 0.60s]
11:42:56 | Finished running 2 view models in 15.13s.


Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2
```

dbt プロジェクトの構築について詳しくは、[クイックスタート ガイド](/guides) を完了することをお勧めします。
