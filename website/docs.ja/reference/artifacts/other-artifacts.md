---
title: "その他のアーティファクトファイル"
sidebar_label: "Other artifacts"
---

### index.html

**作成元:** [`docs generate`](/reference/commands/cmd-docs)

このファイルは、[自動生成された dbt ドキュメント ウェブサイト](/docs/collaborate/build-and-view-your-docs) の骨組みです。サイトの内容は、[マニフェスト](/reference/artifacts/manifest-json) と [カタログ](catalog-json) によって生成されます。

注: `index.json` のソースコードは [dbt-docs リポジトリ](https://github.com/dbt-labs/dbt-docs) から取得されています。ドキュメント サイトに関するバグ報告、提案、または貢献を行う場合は、リポジトリにアクセスしてください。

### partial_parse.msgpack

**生成元:** [マニフェストコマンド](/reference/artifacts/manifest-json) + [`parse`](/reference/commands/parse)

このファイルは、dbt が解析したファイルの圧縮表現を保存するために使用されます。[部分解析](/reference/parsing#partial-parsing) が有効になっている場合、dbt はこのファイルを使用して変更されたファイルを識別し、残りのファイルの再解析を回避します。

### graph.gpickle

**生成元:** [ノード選択](/reference/node-selection/syntax)をサポートするコマンド

dbtリソースDAGのネットワーク表現を格納します。

### graph_summary.json

**生成元:** [マニフェストコマンド](/reference/artifacts/manifest-json)

このファイルは、dbt Core のグラフアルゴリズムにおけるパフォーマンスの問題を調査するのに役立ちます。

[`manifest.json`](/reference/artifacts/manifest-json) や [`graph.gpickle`](#graph.gpickle) よりも匿名化され、コンパクトになっています。

このファイルには、2つの異なる時点の情報が含まれています。
1. `linked` - グラフがリンクされた直後、および
2. `with_test_edges` - テストエッジが追加された直後。

これらの時点には、各ノードの `name` と `type` が含まれ、`succ` には子ノードのキーが含まれます。

