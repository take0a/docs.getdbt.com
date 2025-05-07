---
title: クエリ内の SQL が不正であったり、データベース エラーが発生した場合はどうなりますか?
description: "エラーメッセージとログを使用してデバッグする"
sidebar_label: 'SQLまたはデータベースエラーをデバッグする方法'
id: sql-errors

---


SQL に間違いがある場合、dbt はデータベースが返すエラーを返します。

```shell
$ dbt run --select customers
Running with dbt=1.9.0
Found 3 models, 9 tests, 0 snapshots, 0 analyses, 133 macros, 0 operations, 0 seed files, 0 sources

14:04:12 | Concurrency: 1 threads (target='dev')
14:04:12 |
14:04:12 | 1 of 1 START view model dbt_alice.customers.......................... [RUN]
14:04:13 | 1 of 1 ERROR creating view model dbt_alice.customers................. [ERROR in 0.81s]
14:04:13 |
14:04:13 | Finished running 1 view model in 1.68s.

Completed with 1 error and 0 warnings:

Database Error in model customers (models/customers.sql)
  Syntax error: Expected ")" but got identifier `your-info-12345` at [13:15]
  compiled SQL at target/run/jaffle_shop/customers.sql

Done. PASS=0 WARN=0 ERROR=1 SKIP=0 TOTAL=1
```

このモデルの下流にあるモデルもすべてスキップされます。エラーメッセージと[コンパイル済みSQL](/faqs/Runs/checking-logs)を使用して、エラーをデバッグしてください。
