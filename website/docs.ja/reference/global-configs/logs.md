---
title: "Logs"
id: "logs"
sidebar: "logs"
---

### ログのフォーマット

dbt は、CLI コンソールとログファイルの 2 つの異なる場所にログを出力します。

`LOG_FORMAT` および `LOG_FORMAT_FILE` 設定は、dbt のログのフォーマット方法を指定します。それぞれに、`json`、`text`、`debug` という同じオプションがあります。

<File name='Usage'>

```text
dbt --log-format json run
```

</File>

`text` 形式はコンソール ログのデフォルトであり、単純なタイムスタンプがプレフィックスとして付いたプレーン テキスト メッセージが含まれます:

```
23:30:16  Running with dbt=1.8.0
23:30:17  Registered adapter: postgres=1.8.0
```

`debug` 形式はログ ファイルのデフォルトであり、`text` 形式と同じですが、より詳細なタイムスタンプが付き、各メッセージの [`invocation_id`](/reference/dbt-jinja-functions/invocation_id)、[`thread_id`](/reference/dbt-jinja-functions/thread_id)、および [ログ レベル](/reference/global-configs/logs#log-level) も含まれます:
```
============================== 16:12:08.555032 | 9089bafa-4010-4f38-9b42-564ec9106e07 ==============================
16:12:08.555032 [info ] [MainThread]: Running with dbt=1.8.0
16:12:08.751069 [info ] [MainThread]: Registered adapter: postgres=1.8.0
```

`json` 形式では、<Term id="json" /> 形式で完全に構造化されたログが出力されます:

```json
{"data": {"log_version": 3, "version": "=1.8.0"}, "info": {"category": "", "code": "A001", "extra": {}, "invocation_id": "82131fa0-d2b4-4a77-9436-019834e22746", "level": "info", "msg": "Running with dbt=1.8.0", "name": "MainReportVersion", "pid": 7875, "thread": "MainThread", "ts": "2024-05-29T23:32:54.993336Z"}}
{"data": {"adapter_name": "postgres", "adapter_version": "=1.8.0"}, "info": {"category": "", "code": "E034", "extra": {}, "invocation_id": "82131fa0-d2b4-4a77-9436-019834e22746", "level": "info", "msg": "Registered adapter: postgres=1.8.0", "name": "AdapterRegistered", "pid": 7875, "thread": "MainThread", "ts": "2024-05-29T23:32:56.437986Z"}}
```

`LOG_FORMAT` が明示的に設定されている場合、コンソールとログ ファイルの両方に影響しますが、`LOG_FORMAT_FILE` はログ ファイルにのみ影響します。

<File name='Usage'>

```text
dbt --log-format-file json run
```

</File>

:::tip Tip: 詳細な構造化ログ

`json` 形式の値を `DEBUG` 設定と組み合わせて使用​​することで、豊富なログ情報が生成され、監視ツールにパイプして分析できます:

```text
dbt --debug --log-format json run
```

詳細については、[構造化ログ](/reference/events-logging#structured-logging)を参照してください。

:::

### ログレベル

`LOG_LEVEL` 設定は、コンソールログとファイルログに記録されるイベントの最小重大度を設定します。これは `--debug` フラグよりも柔軟な代替手段です。ログレベルには `debug`、`info`、`warn`、`error`、`none` のいずれかを選択できます。

- `--log-level` を設定すると、コンソールログとファイルログが設定されます。

  ```text
  dbt --log-level debug run
  ```

- `LOG_LEVEL` を `none` に設定すると、コンソールまたはファイル ログへの情報の送信が無効になります。
  
  ```text
  dbt --log-level none
  ```

- ファイル ログ レベルをコンソールとは異なる値に設定するには、`--log-level-file` フラグを使用します。

  ```text
  dbt --log-level-file error run
  ```

- ログ ファイルへの書き込みを無効にしてコンソール ログを保持するには、`LOG_LEVEL_FILE` 構成を none に設定します。

  ```text
  dbt --log-level-file none
  ```

### デバッグレベルのログ出力

`DEBUG` 設定は、dbt のデバッグログを標準出力にリダイレクトします。これにより、`logs/dbt.log` ファイルに加えて、ターミナルにもデバッグレベルのログ情報が表示されます。この出力は詳細です。

`--debug` フラグは、`-d` という短縮形でも使用できます。

<File name='Usage'>

```text
dbt --debug run
...

```

</File>  


### ログとターゲットのパス

デフォルトでは、dbt はログを `logs/` というディレクトリに書き込み、その他のすべてのアーティファクトを `target/` というディレクトリに書き込みます。これらのディレクトリはどちらも、アクティブプロジェクトの `dbt_project.yml` を基準とした相対パスで配置されます。

他のグローバル設定と同様に、CLI オプション (`--target-path`、`--log-path`) または環境変数 (`DBT_TARGET_PATH`、`DBT_LOG_PATH`) を使用して、環境や呼び出しに合わせてこれらの値を上書きできます。

### エラー以外のログを出力から抑制する

デフォルトでは、dbt はすべてのログを標準出力 (stdout) に表示します。`QUIET` 設定を使用すると、エラーログのみを標準出力に表示できます。ログには、[`print()`](/reference/dbt-jinja-functions/print) マクロに渡されたすべての出力が含まれます。例えば、エラーログ以外のログを抑制して、Jinja エラーの検出とデバッグを容易にすることができます。

<File name='profiles.yml'>

```yaml
config:
  quiet: true
```

</File>

エラー ログのみを表示し、エラー以外のログを抑制したい場合は、`dbt run` に `-q` または `--quiet` フラグを指定します。

```text
dbt --quiet run
...
```

### dbt list のログ出力

[dbt バージョン 1.5](/docs/dbt-versions/core-upgrade/Older%20versions/upgrading-to-v1.5#behavior-changes) では、[dbt list](/reference/commands/list) コマンドのログ出力動作が更新され、デフォルトで `INFO` レベルのログが含まれるようになりました。

以下のいずれかのパラメータを使用することで、結果を [`jq`](https://jqlang.github.io/jq/manual/)、ファイル、または別のプロセスにパイプするなど、下流のプロセスと互換性のあるクリーンな出力を得ることができます。

- `dbt --log-level warn list` (推奨。以前のデフォルトと同等)
- `dbt --quiet list` (「印刷」されたメッセージとリスト出力を除き、`ERROR` レベル未満のすべてのログ出力を抑制)


### リレーショナルキャッシュイベントのログ記録

import LogLevel from '/snippets.ja/_log-relational-cache.md';

<LogLevel
event={<a href="https://docs.getdbt.com/reference/global-configs/cache">relational cache</a>}
/>

### 色

ファイルログの色設定は、`profiles.yml` 内、または `--use-colors-file / --no-use-colors-file` フラグを使用してのみ設定できます。

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
