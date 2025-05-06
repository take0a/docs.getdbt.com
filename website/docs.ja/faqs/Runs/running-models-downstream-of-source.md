---
title: 1 つのソースの下流でモデルを実行するにはどうすればよいですか?
description: "ソースセレクターを使用してソースの下流でモデルを実行します"
sidebar_label: '1つのソースの下流でモデルを実行する'
id: running-model-downstream-of-source

---
ソースの下流でモデルを実行するには、`source:` セレクターを使用します。

```shell
$ dbt run --select source:jaffle_shop+
```
(`--select` の代わりに `-s` ショートカットを使用することもできます)

1 つのソース <Term id="table" /> の下流でモデルを実行するには:

```shell
$ dbt run --select source:jaffle_shop.orders+
```

その他の例については、[モデル選択構文](/reference/node-selection/syntax)を確認してください。
