---
title: "dbt snapshot コマンドについて"
sidebar_label: "snapshot"
id: "snapshot"
---

`dbt snapshot` コマンドは、プロジェクトで定義された [スナップショット](/docs/build/snapshots) を実行します。

dbt は、`dbt_project.yml` ファイルで定義された `snapshot-paths` パス内でスナップショットを検索します。デフォルトでは、`snapshot-paths` パスは `snapshots/` です。

**使用法：**

```
$ dbt snapshot --help
usage: dbt snapshot [-h] [--profiles-dir PROFILES_DIR]
                                     [--profile PROFILE] [--target TARGET]
                                     [--vars VARS] [--bypass-cache]
                                     [--threads THREADS]
                                     [--select SELECTOR [SELECTOR ...]]
                                     [--exclude EXCLUDE [EXCLUDE ...]]

optional arguments:
  --select SELECTOR [SELECTOR ...]
                        Specify the snapshots to include in the run.
  --exclude EXCLUDE [EXCLUDE ...]
                        Specify the snapshots to exclude in the run.
```
