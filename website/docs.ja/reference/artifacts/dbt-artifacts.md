---
title: "dbtアーティファクトについて"
sidebar_label: "dbtアーティファクトについて"
---

呼び出しごとに、dbt は 1 つ以上の *アーティファクト* を生成して保存します。これらのうちいくつかは <Term id="json" /> ファイル (`semantic_manifest.json`、`manifest.json`、`catalog.json`、`run_results.json`、`sources.json`) であり、以下の機能を実現するために使用されます。

- [ドキュメント](/docs/collaborate/build-and-view-your-docs)
- [状態](/reference/node-selection/syntax#about-node-selection)
- [ソースの鮮度を視覚化する](/docs/build/sources#source-data-freshness)

これらは以下の目的にも使用できます。

- [dbt セマンティック レイヤー](/docs/use-dbt-semantic-layer/dbt-sl) に関する洞察を得る
- プロジェクトレベルのテスト カバレッジを計算する
- 実行時間の長期分析を実行する
- <Term id="table" /> 構造の履歴的な変化を特定する
- その他、様々なことを行う

### アーティファクトはいつ生成されますか？ <Lifecycle status="team,enterprise"/>

ほとんどの dbt コマンド（および対応する RPC メソッド）は、以下のアーティファクトを生成します。
- [セマンティック マニフェスト](/reference/artifacts/sl-manifest): dbt プロジェクトが解析されるたびに生成されます。
- [マニフェスト](/reference/artifacts/manifest-json): プロジェクトを読み込んで理解するコマンドによって生成されます。
- [実行結果](/reference/artifacts/run-results-json): DAG 内のノードを実行、コンパイル、またはカタログ化するコマンドによって生成されます。
- [カタログ](catalog-json): `docs generate` によって生成されます。
- [ソース](/reference/artifacts/sources-json): `source freshness` によって生成されます。

[dbt Cloud CLI](/docs/cloud/cloud-cli-installation) からコマンドを実行すると、すべてのアーティファクトがデフォルトでダウンロードされます。この動作を変更する場合は、[アーティファクトのダウンロードをスキップする方法](/docs/cloud/configure-cloud-cli#how-to-skip-artifacts-from-being-downloaded)を参照してください。

## アーティファクトはどこに生成されますか？

デフォルトでは、アーティファクトはdbtプロジェクトの `/target` ディレクトリに書き込まれます。[`target-path` フラグ](/reference/global-configs/json-artifacts) を使用して場所を設定できます。

## 共通メタデータ

dbt によって生成されるすべてのアーティファクトには、以下のプロパティを持つ `metadata` ディクショナリが含まれます。

- `dbt_version`: このアーティファクトを生成した dbt のバージョン。リリースのバージョン管理の詳細については、[バージョン管理](/reference/commands/version#versioning) を参照してください。
- `dbt_schema_version`: このアーティファクトのスキーマの URL。下記の注記を参照してください。
- `generated_at`: このアーティファクトが生成された UTC でのタイムスタンプ。
- `adapter_type`: アダプタ（データベース）。例: `postgres`、`spark` など。
- `env`: `DBT_ENV_CUSTOM_ENV_` で始まる環境変数は、プレフィックスを除いた変数名をキーとしてディクショナリに含まれます。
- [`invocation_id`](/reference/dbt-jinja-functions/invocation_id): この dbt 呼び出しの一意の識別子

マニフェストでは、`metadata` に以下の情報も含まれる場合があります。
- `send_anonymous_usage_stats`: この呼び出しが実行中に [匿名使用状況統計](/reference/global-configs/usage-stats) を送信したかどうか。
- `project_name`: ルートプロジェクトの `dbt_project.yml` で定義された `name`。(マニフェスト v10 / dbt Core v1.6 で追加)
- `project_id`: `project_name` からハッシュ化されたプロジェクト識別子。匿名使用状況統計が有効になっている場合は、匿名使用状況統計とともに送信されます。
- `user_id`: ユーザー識別子。デフォルトで `~/dbt/.user.yml` に保存され、匿名使用状況統計が有効になっている場合は、匿名使用状況統計とともに送信されます。

#### 注記:

- dbt アーティファクトの構造は、[JSON スキーマ](https://json-schema.org/) によって標準化されており、[schemas.getdbt.com](https://schemas.getdbt.com/) でホストされています。
- アーティファクトのバージョンは、dbt のマイナーバージョン (`v1.x.0`) で変更される可能性があります。各アーティファクトは個別にバージョン管理されます。

## 関連ドキュメント
- [その他のアーティファクト](/reference/artifacts/other-artifacts) ファイル（`index.html` や `graph_summary.json` など）。
