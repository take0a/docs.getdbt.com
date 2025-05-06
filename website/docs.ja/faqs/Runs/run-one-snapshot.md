---
title: 一度に 1 つのスナップショットを実行するにはどうすればよいですか?
description: "select フラグを使用して、一度に 1 つのスナップショットを実行します。"
sidebar_label: '一度に1つのスナップショットを実行する'
id: run-one-snapshot

---

1 つのスナップショットを実行するには、`--select` フラグの後にスナップショットの名前を指定します:

```shell
$ dbt snapshot --select order_snapshot
```

その他の演算子と例については、[モデル選択構文のドキュメント](/reference/node-selection/syntax)を参照してください。
