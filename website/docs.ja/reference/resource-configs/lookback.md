---
title: "lookback"
id: "lookback"
sidebar_label: "lookback"
resource_types: [models]
description: "Configure `lookback` to determine how many 'batches' of `batch_size` to reprocess when a dbt microbatch incremental model runs incrementally" 
datatype: int
---

<VersionCallout version="1.9" />
## 定義

[マイクロバッチ増分モデル](/docs/build/incremental-microbatch)の実行中に追加バッチを再処理するための`lookback`ウィンドウを設定します。最新のブックマーク（最後に正常に処理されたデータポイント）までXバッチを処理し、遅れて到着したレコードをキャプチャします。

`lookback`には0以上の整数を設定します。デフォルト値は`1`です。[マイクロバッチ増分モデル](/docs/build/incremental-microbatch)の`lookback`は、`dbt_project.yml`ファイル、プロパティYAMLファイル、または設定ブロックで設定できます。

## 例

以下の例では、`user_sessions` モデルの `lookback` 設定として `2` を設定します。

`dbt_project.yml` ファイル内の例:

<File name='dbt_project.yml'>

```yml
models:
  my_project:
    user_sessions:
      +lookback: 2
```
</File>

プロパティ YAML ファイルの例:

<File name='models/properties.yml'>

```yml
models:
  - name: user_sessions
    config:
      lookback: 2
```

</File>

SQL モデル構成ブロックの例:

<File name="models/user_sessions.sql">

```sql
{{ config(
    lookback=2
) }}
```

</File> 
