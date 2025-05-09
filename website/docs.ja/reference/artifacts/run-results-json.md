---
title: "実行結果のJSONファイル"
sidebar_label: "Run results"
---

**現在のスキーマ**: [`v6`](https://schemas.getdbt.com/dbt/run-results/v6/index.html)

**生成元:**
[`build`](/reference/commands/build)
[`clone`](/reference/commands/clone)
[`compile`](/reference/commands/compile)
[`docs generate`](/reference/commands/cmd-docs)
[`retry`](/reference/commands/retry)
[`run`](/reference/commands/run)
[`seed`](/reference/commands/seed)
[`show`](/reference/commands/show)
[`snapshot`](/reference/commands/snapshot)
[`test`](/reference/commands/test)
[`run-operation`](/reference/commands/run-operation)

このファイルには、実行された各ノード（モデル、テストなど）のタイミングとステータス情報を含む、dbt の完了した呼び出しに関する情報が含まれています。複数の `run_results.json` を組み合わせることで、平均モデル実行時間、テスト失敗率、スナップショットによってキャプチャされたレコード変更数などを計算できます。

実行結果には実行されたノードのみが表示されることに注意してください。異なる基準を持つ複数の実行ステップまたはテストステップがある場合、それぞれが異なる実行結果を生成します。

注: `dbt source freshness` は、同様の属性を持つ別のアーティファクト [`sources.json`](/reference/artifacts/sources-json) を生成します。

### 最上位キー

- [`metadata`](/reference/artifacts/dbt-artifacts#common-metadata)
- `args`: このアーティファクトを生成したCLIコマンドまたはRPCメソッドに渡された引数の辞書。最も有用なのは`which` (コマンド)または`rpc_method`です。この辞書はnull値を除外し、nullでない場合はデフォルト値を含めます。dbt-Jinjaコンテキストの[`invocation_args_dict`](/reference/dbt-jinja-functions/flags#invocation_args_dict)に相当します。
- `elapsed_time`: 呼び出し時間の合計（秒）。
- `results`: ノード実行の詳細の配列。

`results` の各エントリは [`Result` オブジェクト](/reference/dbt-classes#result-objects) ですが、1 つの違いがあります。`node` オブジェクト全体ではなく、`unique_id` のみが含まれます。(`node` オブジェクト全体は [`manifest.json`](/reference/artifacts/manifest-json) に記録されます。)

- `unique_id`: 結果を [manifest](/reference/artifacts/manifest-json) 内の `nodes` にマッピングする一意のノード識別子です。
- `status`: dbt による実行時の成功、失敗、またはエラーの解釈です。
- `thread_id`: このノードを実行したスレッド。例: `Thread-1`
- `execution_time`: このノードの実行に費やされた合計時間
- `timing`: 実行時間をステップ（通常は `compile` + `execute`）に分割した配列
- `message`: データベースから返された情報に基づいて、dbt が CLI にこの結果をどのように報告するか

import RowsAffected from '/snippets.ja/_run-result.md';

<RowsAffected/>

<!-- this partial comes from https://github.com/dbt-labs/docs.getdbt.com/tree/current/website/snippets/_run-result-->

run_results.json には、`unique_id` を補完する `applied` 状態に関連する 3 つの属性が含まれています:

- `compiled`: ノードのコンパイル状態を表すブール値エントリ（解析後は `False` ですが、コンパイル後は `True` になります）。
- `compiled_code`: コンパイルされたコードのレンダリングされた文字列（解析後は空ですが、コンパイル後は完全な文字列になります）。
- `relation_name`: データベース内で作成/更新された（または作成/更新される）オブジェクトの完全修飾名。

manifest.json 内の完全なノードオブジェクトを使用して、`unique_id` を介してノードの `logical` 状態に関する追加情報を引き続き検索します。

## 例

以下にいくつかの例と、その結果が `run_results.json` ファイルに出力された結果を示します。

### モデル結果をコンパイルする

次のようなモデルがあるとします:

<File name='models/my_model.sql'>

```sql
select {{ dbt.current_timestamp() }} as created_at
```

</File>

モデルをコンパイルします:

```shell
dbt compile -s my_model
```

以下は `run_results.json` に出力されたスニペットです:

```json
    {
      "status": "success",
      "timing": [
        {
          "name": "compile",
          "started_at": "2023-10-12T16:35:28.510434Z",
          "completed_at": "2023-10-12T16:35:28.519086Z"
        },
        {
          "name": "execute",
          "started_at": "2023-10-12T16:35:28.521633Z",
          "completed_at": "2023-10-12T16:35:28.521641Z"
        }
      ],
      "thread_id": "Thread-2",
      "execution_time": 0.0408780574798584,
      "adapter_response": {},
      "message": null,
      "failures": null,
      "unique_id": "model.my_project.my_model",
      "compiled": true,
      "compiled_code": "select now() as created_at",
      "relation_name": "\"postgres\".\"dbt_dbeatty\".\"my_model\""
    }
```

### 汎用データテストを実行する

[`store_failures_as`](/reference/resource-configs/store_failures_as) 設定を使用して、1 つのデータテストのみの失敗をデータベースに保存します:

<File name='models/_models.yml'>

```yaml
models:
  - name: my_model
    columns:
      - name: created_at
        tests:
          - not_null:
              config:
                store_failures_as: view
          - unique:
              config:
                store_failures_as: ephemeral
```

</File>

組み込みの `unique` テストを実行し、失敗をテーブルとして保存します:

```shell
dbt test -s my_model
```

以下は `run_results.json` から印刷されたスニペットです:

```json
  "results": [
    {
      "status": "pass",
      "timing": [
        {
          "name": "compile",
          "started_at": "2023-10-12T17:20:51.279437Z",
          "completed_at": "2023-10-12T17:20:51.317312Z"
        },
        {
          "name": "execute",
          "started_at": "2023-10-12T17:20:51.319812Z",
          "completed_at": "2023-10-12T17:20:51.441967Z"
        }
      ],
      "thread_id": "Thread-2",
      "execution_time": 0.1807551383972168,
      "adapter_response": {
        "_message": "SELECT 1",
        "code": "SELECT",
        "rows_affected": 1
      },
      "message": null,
      "failures": 0,
      "unique_id": "test.my_project.unique_my_model_created_at.a9276afbbb",
      "compiled": true,
      "compiled_code": "\n    \n    \n\nselect\n    created_at as unique_field,\n    count(*) as n_records\n\nfrom \"postgres\".\"dbt_dbeatty\".\"my_model\"\nwhere created_at is not null\ngroup by created_at\nhaving count(*) > 1\n\n\n",
      "relation_name": null
    },
    {
      "status": "pass",
      "timing": [
        {
          "name": "compile",
          "started_at": "2023-10-12T17:20:51.274049Z",
          "completed_at": "2023-10-12T17:20:51.295237Z"
        },
        {
          "name": "execute",
          "started_at": "2023-10-12T17:20:51.296361Z",
          "completed_at": "2023-10-12T17:20:51.491327Z"
        }
      ],
      "thread_id": "Thread-1",
      "execution_time": 0.22345590591430664,
      "adapter_response": {
        "_message": "SELECT 1",
        "code": "SELECT",
        "rows_affected": 1
      },
      "message": null,
      "failures": 0,
      "unique_id": "test.my_project.not_null_my_model_created_at.9b412fbcc7",
      "compiled": true,
      "compiled_code": "\n    \n    \n\n\n\nselect *\nfrom \"postgres\".\"dbt_dbeatty\".\"my_model\"\nwhere created_at is null\n\n\n",
      "relation_name": "\"postgres\".\"dbt_dbeatty_dbt_test__audit\".\"not_null_my_model_created_at\""
    }
  ],
```
