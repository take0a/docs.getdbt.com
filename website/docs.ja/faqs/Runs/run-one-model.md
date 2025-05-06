---
title: 一度に 1 つのモデルを実行するにはどうすればよいですか?
description: "select フラグを使用して、一度に 1 つのモデルを実行します。"
sidebar_label: '一度に1つのモデルを実行する'
id: run-one-model

---

1 つのモデルを実行するには、`--select` フラグ (または `-s` フラグ) の後にモデルの名前を指定します:

```shell
$ dbt run --select customers
```

その他の演算子と例については、[モデル選択構文のドキュメント](/reference/node-selection/syntax)を参照してください:
