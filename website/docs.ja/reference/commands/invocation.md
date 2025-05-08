---
title: "dbt invocation コマンドについて"
sidebar_label: "invocation"
id: invocation
---

`dbt invocation` コマンドは [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) で使用でき、次の操作を実行できます。
- アクティブな呼び出しを一覧表示して、長時間実行されている呼び出しやハングしている呼び出しをデバッグします。
- `Session supplied` エラーの原因となっているセッションを特定して調査します。
- 現在アクティブな dbt コマンド (`run`、`build` など) をリアルタイムで監視します。

`dbt invocation` コマンドは、_アクティブな呼び出し_ のみを一覧表示します。実行中のセッションがない場合、リストは空になります。完了したセッションは出力に含まれません。

## 使用方法

このページでは、`dbt invocation` で使用できるコマンドとフラグの一覧を示します。これらを使用するには、`dbt invocation [command]` のようにコマンドまたはオプションを追加します。

コマンドラインインターフェース (CLI) で使用できるフラグは、[`help`](#dbt-invocation-help) と [`list`](#dbt-invocation-list) です。

### dbt invocation help

`help` コマンドは、使用可能なフラグを含む、CLI の `invocation` コマンドのヘルプ出力を提供します。

```shell
dbt invocation help
```

or

```shell
dbt help invocation
```

このコマンドは次の情報を返します。

```bash
dbt invocation help
Manage invocations

Usage:
  dbt invocation [command]

Available Commands:
  list        List active invocations

Flags:
  -h, --help   help for invocation

Global Flags:
      --log-format LogFormat   The log format, either json or plain. (default plain)
      --log-level LogLevel     The log level, one of debug, info, warning, error or fatal. (default info)
      --no-color               Disables colorization of the output.
  -q, --quiet                  Suppress all non-error logging to stdout.

Use "dbt invocation [command] --help" for more information about a command.
```

### dbt invocation list

`list` コマンドは、dbt Cloud CLI でアクティブな呼び出しのリストを表示します。長時間実行セッションがアクティブな場合は、別のターミナルウィンドウでこのコマンドを使用してアクティブなセッションを表示し、問題のデバッグに役立てることができます。

```shell
dbt invocation list
```

このコマンドは、アクティブ セッションの `ID`、`status`、`type`、`arguments`、`started at` 時間など、次の情報を返します:

```bash
dbt invocation list

Active Invocations:
  ID                             6dcf4723-e057-48b5-946f-a4d87e1d117a
  Status                         running
  Type                           cli
  Args                           [run --select test.sql]
  Started At                     2025-01-24 11:03:19

➜  jaffle-shop git:(test-cli) ✗ 
```

:::tip

ターミナルでアクティブなセッションをキャンセルするには、`Ctrl + Z` ショートカットを使用します。

:::

## 関連ドキュメント

- [dbt Cloud CLI のインストール](/docs/cloud/cloud-cli-installation)
- [dbt Cloud CLI の「セッションが占有されています」エラーのトラブルシューティング](/faqs/Troubleshooting/long-sessions-cloud-cli)


