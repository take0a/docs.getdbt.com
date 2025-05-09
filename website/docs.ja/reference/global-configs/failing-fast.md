---
title: "早く失敗する"
id: "failing-fast"
sidebar: "早く失敗する"
---

`dbt run` に `-x` または `--fail-fast` フラグを指定すると、1 つのリソースのビルドに失敗した場合、dbt は直ちに終了します。最初のモデルが失敗したときに他のモデルが実行中だった場合、dbt はこれらの実行中のモデルの接続を終了します。

例えば、実行するモデルを 4 つ選択し、最初のモデルでエラーが発生すると、そのエラーによって他のモデルは実行されなくなります。

```text
dbt -x run --threads 1
Running with dbt=1.0.0
Found 4 models, 1 test, 1 snapshot, 2 analyses, 143 macros, 0 operations, 1 seed file, 0 sources

14:47:39 | Concurrency: 1 threads (target='dev')
14:47:39 |
14:47:39 | 1 of 4 START table model test_schema.model_1........... [RUN]
14:47:40 | 1 of 4 ERROR creating table model test_schema.model_1.. [ERROR in 0.06s]
14:47:40 | 2 of 4 START view model test_schema.model_2............ [RUN]
14:47:40 | CANCEL query model.debug.model_2....................... [CANCEL]
14:47:40 | 2 of 4 ERROR creating view model test_schema.model_2... [ERROR in 0.05s]

Database Error in model model_1 (models/model_1.sql)
  division by zero
  compiled SQL at target/run/debug/models/model_1.sql

Encountered an error:
FailFast Error in model model_1 (models/model_1.sql)
  Failing early due to test failure or runtime error
```
