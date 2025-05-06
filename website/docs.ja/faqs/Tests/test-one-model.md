---
title: 一度に 1 つのモデルをテストするにはどうすればよいですか?
description: "select フラグを使用して、一度に 1 つのモデルをテストします。"
sidebar_label: '一度に1つのモデルをテストする'
id: test-one-model

---

1 つのモデルでテストを実行するのは、モデルを実行するのと非常に似ています。`--select` フラグ (または `-s` フラグ) を使用し、その後にモデルの名前を指定します:

```shell
dbt test --select customers
```

完全な構文については[モデル選択構文のドキュメント](/reference/node-selection/syntax)を、特に[テスト選択の例](/reference/node-selection/test-selection-examples)を確認してください。
