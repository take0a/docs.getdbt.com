---
title: シードの下流でモデルを実行するにはどうすればよいですか?
description: "モデル選択構文を使用してモデルを下流で実行する"
sidebar_label: 'シードの下流でモデルを実行する'
id: run-downstream-of-seed

---

[モデル選択構文](/reference/node-selection/syntax)を使用して、シードをモデルのように扱い、シードの下流でモデルを実行できます。

例えば、以下のコマンドは、`country_codes` という名前のシードの下流ですべてのモデルを実行します。

```shell
$ dbt run --select country_codes+
```
