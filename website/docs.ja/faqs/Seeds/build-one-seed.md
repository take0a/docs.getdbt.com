---
title: 一度に 1 つのシードを構築するにはどうすればよいでしょうか?
description: "select フラグを使用して、一度に 1 つのシードを構築します。"
sidebar_label: "一度に1つのシードを構築する"
id: build-one-seed
---

次のように、`dbt seed` コマンドで `--select` オプションを使用できます。

```shell

$ dbt seed --select country_codes

```

`--exclude` オプションもあります。

[モデル選択構文](/reference/node-selection/syntax) のドキュメントで詳細をご確認ください。

