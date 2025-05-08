---
title: "dbt retry コマンドについて"
sidebar_label: "retry"
id: "retry"
---

`dbt retry` は、ノードの障害発生時点から最後に実行した `dbt` コマンドを再実行します。前回実行した `dbt` コマンドが成功した場合、`retry` は `no action` として終了します。

再試行は、以下のコマンドで機能します:

- [`build`](/reference/commands/build)
- [`compile`](/reference/commands/compile)
- [`clone`](/reference/commands/clone)
- [`docs generate`](/reference/commands/cmd-docs#dbt-docs-generate)
- [`seed`](/reference/commands/seed)
- [`snapshot`](/reference/commands/build)
- [`test`](/reference/commands/test)
- [`run`](/reference/commands/run)
- [`run-operation`](/reference/commands/run-operation)

`dbt retry` は [run_results.json](/reference/artifacts/run-results-json) を参照して実行開始位置を決定します。以前の失敗を修正せずに `dbt retry` を実行すると、<Term id="idempotent" /> の結果が取得されます。

`dbt retry` は、以前実行したコマンドの [セレクター](/reference/node-selection/yaml-selectors) を再利用します。

`dbt run` が成功した後に `dbt retry` を実行した場合の結果の例:

```shell
Running with dbt=1.6.1
Registered adapter: duckdb=1.6.0
Found 5 models, 3 seeds, 20 tests, 0 sources, 0 exposures, 0 metrics, 348 macros, 0 groups, 0 semantic models
 
Nothing to do. Try checking your model configs and model specification args
```

`dbt run` がモデル内で構文エラーに遭遇した場合の例:

```shell
Running with dbt=1.6.1
Registered adapter: duckdb=1.6.0
Found 5 models, 3 seeds, 20 tests, 0 sources, 0 exposures, 0 metrics, 348 macros, 0 groups, 0 semantic models

Concurrency: 24 threads (target='dev')
 
1 of 5 START sql view model main.stg_customers ................................. [RUN]
2 of 5 START sql view model main.stg_orders .................................... [RUN]
3 of 5 START sql view model main.stg_payments .................................. [RUN]
1 of 5 OK created sql view model main.stg_customers ............................ [OK in 0.06s]
2 of 5 OK created sql view model main.stg_orders ............................... [OK in 0.06s]
3 of 5 OK created sql view model main.stg_payments ............................. [OK in 0.07s]
4 of 5 START sql table model main.customers .................................... [RUN]
5 of 5 START sql table model main.orders ....................................... [RUN]
4 of 5 ERROR creating sql table model main.customers ........................... [ERROR in 0.03s]
5 of 5 OK created sql table model main.orders .................................. [OK in 0.04s]
 
Finished running 3 view models, 2 table models in 0 hours 0 minutes and 0.15 seconds (0.15s).
  
Completed with 1 error and 0 warnings:
  
Runtime Error in model customers (models/customers.sql)
 Parser Error: syntax error at or near "selct"

Done. PASS=4 WARN=0 ERROR=1 SKIP=0 TOTAL=5
```


エラーを修正せずに `dbt retry` 実行が失敗した後続の例:

```shell
Running with dbt=1.6.1
Registered adapter: duckdb=1.6.0
Found 5 models, 3 seeds, 20 tests, 0 sources, 0 exposures, 0 metrics, 348 macros, 0 groups, 0 semantic models

Concurrency: 24 threads (target='dev')

1 of 1 START sql table model main.customers .................................... [RUN]
1 of 1 ERROR creating sql table model main.customers ........................... [ERROR in 0.03s]

Done. PASS=4 WARN=0 ERROR=1 SKIP=0 TOTAL=5
```

エラーを修正した後の `dbt retry` 実行の成功例:

```shell
Running with dbt=1.6.1
Registered adapter: duckdb=1.6.0
Found 5 models, 3 seeds, 20 tests, 0 sources, 0 exposures, 0 metrics, 348 macros, 0 groups, 0 semantic models
 
Concurrency: 24 threads (target='dev')

1 of 1 START sql table model main.customers .................................... [RUN]
1 of 1 OK created sql table model main.customers ............................... [OK in 0.05s]

Finished running 1 table model in 0 hours 0 minutes and 0.09 seconds (0.09s).
 
Completed successfully
  
Done. PASS=1 WARN=0 ERROR=0 SKIP=0 TOTAL=1
```

それぞれのシナリオにおいて、`dbt retry` は上流の依存関係をすべて再度実行するのではなく、エラーから回復します。
