---
title: "イベントとログ"
---

dbt の実行中、イベントが生成されます。これらのイベントを確認する最も一般的な方法は、以下の 2 つの場所にリアルタイムで書き込まれるログメッセージです。
- コマンドラインターミナル (`stdout`)。dbt 実行中にインタラクティブなフィードバックを提供します。
- デバッグログファイル (`logs/dbt.log`)。エラー発生時に詳細な [デバッグ](/guides/debug-errors) を可能にします。このファイル内のテキスト形式のログメッセージには、すべての `DEBUG` レベルのイベントに加え、ログレベルやスレッド名などのコンテキスト情報が含まれます。このファイルの場所は、[`log-path` フラグ](/reference/global-configs/logs) で設定できます。

<File name='CLI'>

```bash
21:35:48  6 of 7 OK created view model dbt_testing.name_list......................... [CREATE VIEW in 0.17s]
```

</File>

<File name='logs/dbt.log'>

```text
============================== 21:21:15.272780 | 48cef052-3819-4550-a83a-4a648aef5a31 ==============================
21:21:15.272780 [info ] [MainThread]: Running with dbt=1.5.0-b5
21:21:15.273802 [debug] [MainThread]: running dbt with arguments {'printer_width': '80', 'indirect_selection': 'eager', 'log_cache_events': 'False', 'write_json': 'True', 'partial_parse': 'True', 'cache_selected_only': 'False', 'warn_error': 'None', 'fail_fast': 'False', 'debug': 'False', 'log_path': '/Users/jerco/dev/scratch/testy/logs', 'profiles_dir': '/Users/jerco/.dbt', 'version_check': 'False', 'use_colors': 'False', 'use_experimental_parser': 'False', 'no_print': 'None', 'quiet': 'False', 'log_format': 'default', 'static_parser': 'True', 'introspect': 'True', 'warn_error_options': 'WarnErrorOptions(include=[], exclude=[])', 'target_path': 'None', 'send_anonymous_usage_stats': 'True'}
21:21:16.190990 [debug] [MainThread]: Partial parsing enabled: 0 files deleted, 0 files added, 0 files changed.
21:21:16.191404 [debug] [MainThread]: Partial parsing enabled, no changes found, skipping parsing
21:21:16.207330 [info ] [MainThread]: Found 2 models, 0 tests, 0 snapshots, 1 analysis, 535 macros, 0 operations, 1 seed file, 0 sources, 0 exposures, 0 metrics, 0 groups

```

</File>

## 構造化ログ

_dbt-core におけるイベントシステムの実装方法の詳細については、[`events` モジュールの README](https://github.com/dbt-labs/dbt-core/blob/HEAD/core/dbt/events/README.md) をご覧ください。_

`dbt-core` の各イベントの構造は、[プロトコルバッファ](https://developers.google.com/protocol-buffers) を使用して定義されたスキーマに基づいています。すべてのスキーマは、`dbt-core` コードベース内の [`types.proto`](https://github.com/dbt-labs/dbt-core/blob/3bf148c443e6b1da394b62e88a08f1d7f1d8ccaa/core/dbt/events/core_types.proto) ファイルで定義されています。

すべてのイベントには、同じ2つのトップレベルキーがあります。
- `info`: すべてのイベントに共通する情報。内訳については、以下の表をご覧ください。
- `data`: このイベントに固有の追加の構造化データ。このイベントがdbtプロジェクト内の特定のノードに関連する場合、共通属性を持つ`node_info`ディクショナリが含まれます。

### `info` fields

| Field       | Description   |
|-------------|---------------|
| `category` | 将来使用するためのプレースホルダー（[dbt-labs/dbt-core#5958](https://github.com/dbt-labs/dbt-core/issues/5958) を参照） |
| `code` | このイベントタイプの一意の短縮識別子（例：`A123`） |
| `extra` |`DBT_ENV_CUSTOM_ENV_` で始まる環境変数に基づくカスタム環境メタデータの辞書 |
| [`invocation_id`](/reference/dbt-jinja-functions/invocation_id) | この dbt 呼び出しの一意の識別子 |
| `level` |ログ レベルの文字列表現 (`debug`、`info`、`warn`、`error`) |
| `log_version` | バージョンを示す整数 |
| `msg` | 構造化された「データ」から構築された、人間が理解しやすいログメッセージです。**注**: このメッセージは機械による処理を想定したものではありません。ログメッセージは、dbt の将来のバージョンで変更される可能性があります。 |
| `name` | このイベントタイプの一意の名前（プロトスキーマ名と一致する） |
| `pid` | このログメッセージを生成した実行中の dbt 呼び出しのプロセス ID |
| `thread_name` | ログ メッセージが生成されたスレッド。dbt が複数のスレッドで実行されるときにクエリを追跡するのに役立ちます。|
| `ts` | ログラインが印刷されたとき |

### `node_info` フィールド

特定の DAG ノード（モデル、シード、テストなど）のコンパイル中または実行中に、多くのイベントが発生します。`node_info` オブジェクトが利用可能な場合、以下の情報が含まれます:

| Field       | Description   |
|-------------|---------------|
| `materialized` | ビュー、テーブル、増分など。 |
| `meta` | このノードのユーザー設定の [`meta` 辞書](/reference/resource-configs/meta) |
| `node_finished_at` | ノード処理が完了したときのタイムスタンプ |
| `node_name` | このモデル/シード/テスト/その他の名前 |
| `node_path` | このリソースが定義されているファイルパス |
| `node_relation` | このノードのデータベース表現を含むネストされたオブジェクト: `database`、`schema`、`alias`、および引用と包含ポリシーが適用された完全な `relation_name` |
| `node_started_at` | ノード処理が開始されたタイムスタンプ |
| `node_status` | ノードの現在のステータス。[結果コントラクト](https://github.com/dbt-labs/dbt-core/blob/eba90863ed4043957330ea44ca267db1a2d81fcd/core/dbt/contracts/results.py#L75-L88)で定義されている「RunningStatus」（実行中）または「NodeStatus」（終了）のいずれかです。 |
| `resource_type` | `model`、`test`、`seed`、`snapshot` など。 |
| `unique_id` | このリソースの一意の識別子。これを使用して、[マニフェスト](/reference/artifacts/manifest-json) 内のより詳細なコンテキスト情報を検索できます。 |

### 例

```json
{
  "data": {
    "description": "sql view model dbt_jcohen.my_model",
    "index": 1,
    "node_info": {
      "materialized": "view",
      "meta": {
        "first": "some_value",
        "second": "1234"
      },
      "node_finished_at": "",
      "node_name": "my_model",
      "node_path": "my_model.sql",
      "node_relation": {
        "alias": "my_model",
        "database": "my_database",
        "relation_name": "\"my_database\".\"my_schema\".\"my_model\"",
        "schema": "my_schema"
      },
      "node_started_at": "2023-04-12T19:27:27.435364",
      "node_status": "started",
      "resource_type": "model",
      "unique_id": "model.my_dbt_project.my_model"
    },
    "total": 1
  },
  "info": {
    "category": "",
    "code": "Q011",
    "extra": {
      "my_custom_env_var": "my_custom_value"
    },
    "invocation_id": "206b4e61-8447-4af7-8035-b174ab3ac991",
    "level": "info",
    "msg": "1 of 1 START sql view model my_database.my_model ................................ [RUN]",
    "name": "LogStartLine",
    "pid": 95894,
    "thread": "Thread-1",
    "ts": "2023-04-12T19:27:27.436283Z"
  }
}
```

## Python インターフェース

以前のバージョンの `dbt-core` では、呼び出し中に発生したイベントの完全な履歴を `EVENT_HISTORY` オブジェクトの形式で利用できました。

[dbt をプログラムで呼び出す](programmatic-invocations#registering-callbacks) 場合、dbt の `EventManager` にコールバックを登録できます。これにより、構造化イベントに Python オブジェクトとしてアクセスできるようになり、カスタムログ記録や他のシステムとの統合が可能になります。

イベントへの Python インターフェースは、構造化ログ記録インターフェースに比べて大幅に未成熟です。標準的なユースケースでは、JSON 形式のログを解析することをお勧めします。