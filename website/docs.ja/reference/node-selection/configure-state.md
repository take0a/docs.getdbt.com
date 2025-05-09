---
title: "state での選択の構成"
description: "dbt で状態選択を構成する方法を学習します。"
pagination_next: "reference/node-selection/state-comparison-caveats"
---

状態と[defer](/reference/node-selection/defer)は、環境変数とCLIフラグで設定できます。

- `--state` または `DBT_STATE`: ファイルパス
- `--defer` または `DBT_DEFER`: ブール値
- `--defer-state` または `DBT_DEFER_STATE`: 遅延処理のみに使用するファイルパス（オプション）

`--defer-state` が指定されていない場合、遅延処理は `--state` によって提供されるアーティファクトを使用します。これにより、ある環境または過去の時点の論理状態と比較し、別の環境または時点の適用済み状態に遅延処理を延期する場合に、よりきめ細かな制御が可能になります。

フラグと環境変数の両方が指定されている場合、フラグが優先されます。

#### 注意事項
- `--state` アーティファクトは、現在実行中の dbt バージョンと互換性のあるスキーマバージョンである必要があります。
- これらは強力かつ複雑な機能です。状態比較については、[既知の注意事項と制限事項](/reference/node-selection/state-comparison-caveats) をご覧ください。

:::warning 構文の非推奨化

[dbt v1.5](/docs/dbt-versions/core-upgrade/Older%20versions/upgrading-to-v1.5#behavior-changes) では、state (`DBT_ARTIFACT_STATE_PATH`) と defer (`DBT_DEFER_TO_STATE`) の旧構文が非推奨となりました。dbt は旧構文との下位互換性をサポートしていますが、将来のリリースで削除される予定です。

:::

### 「結果」ステータス

ジョブ状態のもう1つの要素は、以前のdbt呼び出しの「結果」です。たとえば、「dbt run」を実行すると、dbtは「run_results.json」アーティファクトを作成します。このアーティファクトには、dbtモデルの実行時間と成功/エラーステータスが含まれます。「run_results.json」の詳細については、[「実行結果」](/reference/artifacts/run-results-json) ページをご覧ください。

以下のdbtコマンドは、「ru​​n_results.json」アーティファクトを生成します。その結果は、後続のdbt呼び出しで参照できます。
- `dbt run`
- `dbt test`
- `dbt build`
- `dbt seed`

上記のコマンドのいずれかを実行した後、次のように後続のコマンドにセレクタを追加することで、結果を参照できます。

```bash
# You can also set the DBT_STATE environment variable instead of the --state flag.
dbt run --select "result:<status>" --defer --state path/to/prod/artifacts
```

利用可能なオプションは、リソース (ノード) の種類によって異なります:

|      `result:\<status>`        | model | seed | snapshot | test |
|----------------|-------|------|------|----------|
| `result:error`   | ✅  | ✅   | ✅   |  ✅      |
| `result:success` | ✅  | ✅   | ✅   |          |
| `result:skipped` | ✅  |      | ✅   |  ✅      |
| `result:fail`    |     |      |      |  ✅      |
| `result:warn`    |     |      |      |  ✅      |
| `result:pass`    |     |      |      |  ✅      |

### `state` セレクタと `result` セレクタの組み合わせ

状態セレクタと結果セレクタを dbt の 1 回の呼び出しで組み合わせて、前回の実行時や新規モデル、あるいは変更されたモデルのエラーを取得することもできます。

```bash
dbt run --select "result:<status>+" state:modified+ --defer --state ./<dbt-artifact-path>
```

### 「source_status」ステータス

ジョブの状態を表すもう1つの要素は、以前のdbt呼び出しの「source_status」です。例えば、「dbt source freshness」を実行すると、dbtは「sources.json」アーティファクトを作成します。このアーティファクトには、dbtソースの実行時間と「max_loaded_at」日付が含まれます。「sources.json」の詳細については、['sources'](/reference/artifacts/sources-json) ページをご覧ください。

「dbt source freshness」コマンドは「sources.json」アーティファクトを生成します。このアーティファクトの結果は、以降のdbt呼び出しで参照できます。

ジョブを選択すると、dbt Cloudはそのジョブの最新の成功した実行のアーティファクトを表示します。dbtはこれらのアーティファクトを使用して、新しいソースのセットを決定します。ジョブコマンドに `source_status:fresher+` 引数を含めることで、dbt に最新のソースとその子のみを実行およびテストするように指示できます。この場合、以前の状態と現在の状態の両方で `sources.json` アーティファクトが利用可能である必要があります。つまり、両方のジョブ状態で `dbt source freshness` を実行する必要があります。

`dbt source freshness` コマンドを実行した後、後続のコマンドにセレクターを追加することで、ソースの鮮度結果を参照できます。

```bash
# You can also set the DBT_STATE environment variable instead of the --state flag.
dbt source freshness # must be run again to compare current to previous state
dbt build --select "source_status:fresher+" --state path/to/prod/artifacts
```
その他のコマンド例については、[ワークフローに関するプロのヒント](/best-practices/best-practice-workflows#pro-tips-for-workflows)を参照してください。

## 関連ドキュメント
- [dbt における状態について](/reference/node-selection/state-selection)
- [状態比較に関する注意事項](/reference/node-selection/state-comparison-caveats)
