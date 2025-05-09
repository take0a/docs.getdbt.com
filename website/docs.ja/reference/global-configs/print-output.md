---
title: "Print output"
id: "print-output"
sidebar: "Print output"
---

### 標準出力（stdout）の `print()` メッセージを抑制

デフォルトでは、dbt は [`print()`](/reference/dbt-jinja-functions/print) メッセージを標準出力（stdout）に出力します。`DBT_PRINT` 環境変数を使用すると、これらのメッセージが標準出力に表示されないようにすることができます。

:::warning 構文の非推奨

dbt v1.5 以降、従来の `DBT_NO_PRINT` 環境変数は非推奨となりました。下位互換性は維持されますが、将来のリリース（現時点では未定）で削除される予定です。

:::

`print()` メッセージが stdout に表示されないようにするには、`dbt run` に `--no-print` フラグを指定します。

```text
dbt --no-print run
```

### プリンタ幅

デフォルトでは、dbt は行を 80 文字幅にパディングして出力します。この設定を変更するには、`profiles.yml` ファイルに以下のコードを追加します:

<File name='profiles.yml'>

```yaml
config:
  printer_width: 120
```

</File>

### 印刷色

デフォルトでは、dbt はターミナルに出力する出力を色分けします。これを無効にするには、`profiles.yml` ファイルに以下のコードを追加します:

<File name='profiles.yml'>

```yaml
config:
  use_colors: False
```

</File>

```text
dbt --use-colors run
dbt --no-use-colors run
```

ファイル ログの色設定は、`profiles.yml` 内、または `--use-colors-file / --no-use-colors-file` フラグを使用してのみ設定できます。

<File name='profiles.yml'>

```yaml
config:
  use_colors_file: False
```

</File>

```text
dbt --use-colors-file run
dbt --no-use-colors-file run
```
