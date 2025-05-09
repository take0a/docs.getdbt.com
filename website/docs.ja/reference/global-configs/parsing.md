---
title: "Parsing"
id: "parsing"
sidebar: "Parsing"
---

### 部分解析

`PARTIAL_PARSE` 設定を使用すると、プロジェクト内で部分解析を有効または無効にできます。詳細については、[解析に関するドキュメント](/reference/parsing#partial-parsing)をご覧ください。

<File name='profiles.yml'>

```yaml

config:
  partial_parse: true

```

</File>

<File name='Usage'>

```text
dbt --no-partial-parse run
```

</File>

### 静的パーサー

`STATIC_PARSER` 設定で、静的パーサーの使用を有効または無効にできます。詳細については、[パーシングに関するドキュメント](/reference/parsing#static-parser)を参照してください。

<File name='profiles.yml'>

```yaml

config:
  static_parser: true

```

</File>

### 試験的なパーサー

現在は使用されていません。
