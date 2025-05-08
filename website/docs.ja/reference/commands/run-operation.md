---
title: "dbt run-operation コマンドについて"
sidebar_label: "run-operation"
description: "dbt の run-operation コマンドを使用してマクロを呼び出す方法については、このガイドをお読みください。"
id: "run-operation"
---

### 概要

`dbt run-operation` コマンドはマクロを呼び出すために使用されます。使用方法については、[operations](/docs/build/hooks-operations#about-operations) のドキュメントをご覧ください。

### 使用法
```
$ dbt run-operation {macro} --args '{args}'
  {macro}        Specify the macro to invoke. dbt will call this macro
                        with the supplied arguments and then exit
  --args ARGS           Supply arguments to the macro. This dictionary will be
                        mapped to the keyword arguments defined in the
                        selected macro. This argument should be a YAML string,
                        eg. '{my_variable: my_value}'
```
### コマンドラインの例

例 1:

`$ dbt run-operation grant_select --args '{role: reporter}'`

例 2:

`$ dbt run-operation clean_stale_models --args '{days: 7, dry_run: True}'`
