---
title: dbt seed コマンドについて"
sidebar_label: "seed"
id: "seed"
---

`dbt seed` コマンドは、dbt プロジェクトの `seed-paths` ディレクトリにある `csv` ファイルを <Term id="data-warehouse" /> に読み込みます。


### 実行するシードの選択

`dbt seed` に `--select` フラグを指定すると、特定のシードを実行できます。例:

```
$ dbt seed --select "country_codes"
Found 2 models, 3 tests, 0 archives, 0 analyses, 53 macros, 0 operations, 2 seed files

14:46:15 | Concurrency: 1 threads (target='dev')
14:46:15 |
14:46:15 | 1 of 1 START seed file analytics.country_codes........................... [RUN]
14:46:15 | 1 of 1 OK loaded seed file analytics.country_codes....................... [INSERT 3 in 0.01s]
14:46:16 |
14:46:16 | Finished running 1 seed in 0.14s.

```
