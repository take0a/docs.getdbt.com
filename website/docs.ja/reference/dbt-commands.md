---
title: "dbt コマンドリファレンス"
---

dbt は以下のツールを使用して実行できます:

- ブラウザで [dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用する
- コマンドラインインターフェースで [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) またはオープンソースの [dbt Core](/docs/core/installation-overview) を使用する

上記のツールとの主な違いは、dbt Cloud CLI と IDE は、dbt Cloud のインフラストラクチャとその包括的な [機能](/docs/cloud/about-cloud/dbt-cloud-features) を活用して、dbt コマンドの安全な並列実行をサポートするように設計されていることです。一方、`dbt-core` は、同一プロセス内での複数の呼び出しの安全な並列実行をサポートしていません。詳しくは、[並列実行](#parallel-execution) セクションをご覧ください。

## 並列実行 {#parallel-execution}

dbt Cloud ではコマンドの同時実行が可能で、データの整合性を損なうことなく効率性を高めることができます。これにより、複数のコマンドを同時に実行できます。ただし、どのコマンドが並列実行可能で、どのコマンドが並列実行不可能かを理解することが重要です。

一方、[`dbt-core` は、同一プロセス内での複数の呼び出しの安全な並列実行を_サポート_していません](/reference/programmatic-invocations#parallel-execution-not-supported)。そのため、ユーザーはデータの整合性とシステムの安定性を確保するために、同時実行を手動で管理する必要があります。

dbt ワークフローの効率性と安全性を確保するために、異なる種類の dbt コマンドを同時に（並列に）実行できます。たとえば、`dbt build`（書き込み操作）と `dbt parse`（読み取り操作）を同時に安全に実行できます。ただし、`dbt build` と `dbt run`（どちらも書き込み操作）を同時に実行することはできません。

dbt コマンドは `read` コマンドまたは `write` コマンドになります:

| Command type | Description | <div style={{width:'200px'}}>Example</div> |
|------|-------------|---------|
| **Write** | これらのコマンドは、データ プラットフォーム内のデータまたはメタデータを変更するアクションを実行します。<br /><br /> 一度に 1 回の呼び出しに制限されているため、データ プラットフォーム内の同じテーブルを同時に上書きするなどの潜在的な競合を回避できます。 | `dbt build`<br />`dbt run` |
| **Read** | これらのコマンドは、データプラットフォームに変更を加えることなく、データの取得または読み取りを行う操作です。<br /><br /> 複数の呼び出しを並行して実行でき、一度に1つの呼び出しに限定されません。つまり、読み取りコマンドは、他の読み取りコマンドや単一の書き込みコマンドと並行して実行できます。| `dbt parse`<br />`dbt compile`|

## 利用可能なコマンド

以下のセクションでは、dbt でサポートされているコマンドとその関連フラグについて説明します。特に記載がない限り、これらのコマンドはすべてのツールとすべての[サポート対象バージョン](/docs/dbt-versions/core)で使用できます。これらのコマンドの前に「dbt」を付けることで、特定のツールで実行できます。たとえば、「test」コマンドを実行するには、「dbt test」と入力します。

コマンドラインでのモデル選択については、[モデル選択構文](/reference/node-selection/syntax)を参照してください。

('❌') が付いたコマンドは書き込みコマンド、('✅') が付いたコマンドは読み取りコマンド、(N/A) が付いたコマンドは dbt コマンドの並列化に関係しないことを示します。

| Command | Description | Parallel execution | <div style={{width:'250px'}}>注意点</div> |
|---------|-------------| :-----------------:| ------------------------------------------ |
| [build](/reference/commands/build) | 選択したすべてのリソース（モデル、シード、スナップショット、テスト）をビルドしてテストします |  ❌ | All tools <br /> All [supported versions](/docs/dbt-versions/core) | 
| cancel | 最新の呼び出しをキャンセルします。 | N/A | dbt Cloud CLI <br /> Requires [dbt v1.6 or higher](/docs/dbt-versions/core) |
| [clean](/reference/commands/clean) | dbt プロジェクトに存在するアーティファクトを削除します |  ✅ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [clone](/reference/commands/clone) | 指定された状態から選択したモデルを複製します |  ❌ | All tools <br /> Requires [dbt v1.6 or higher](/docs/dbt-versions/core) |
| [compile](/reference/commands/compile) | プロジェクト内のモデルをコンパイルします（実行はしません）。 |  ✅ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [debug](/reference/commands/debug) | dbt接続とプロジェクトをデバッグします | ✅ | dbt Cloud IDE, dbt Cloud CLI, dbt Core <br /> All [supported versions](/docs/dbt-versions/core) |
| [deps](/reference/commands/deps) | プロジェクトの依存関係をダウンロードします |  ✅ |  All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [docs](/reference/commands/cmd-docs) | プロジェクトのドキュメントを生成する |   ✅ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [environment](/reference/commands/dbt-environment) |dbt Cloud 環境と対話できるようになります。 |   N/A | dbt Cloud CLI <br /> Requires [dbt v1.5 or higher](/docs/dbt-versions/core) |
| help | 任意のコマンドのヘルプ情報を表示します | N/A | dbt Core, dbt Cloud CLI <br /> All [supported versions](/docs/dbt-versions/core) |
| [init](/reference/commands/init) | 新しいdbtプロジェクトを初期化します |   ✅ | dbt Core<br /> All [supported versions](/docs/dbt-versions/core) |
| [invocation](/reference/commands/invocation) | アクティブな呼び出しを操作して、長時間実行されるセッションをユーザーがデバッグできるようにします。 |  N/A | dbt Cloud CLI<br /> Requires [dbt v1.5 or higher](/docs/dbt-versions/core) |
| [list](/reference/commands/list) | dbt プロジェクトで定義されたリソースを一覧表示します |  ✅ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [parse](/reference/commands/parse) | プロジェクトを解析し、詳細なタイミング情報を書き込みます |  ✅ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| reattach | 最新の呼び出しに再接続して、ログと成果物を取得します。 |   N/A | dbt Cloud CLI <br /> Requires [dbt v1.6 or higher](/docs/dbt-versions/core) |
| [retry](/reference/commands/retry) | 最後に実行した`dbt`コマンドを失敗した時点から再試行します |  ❌ | All tools <br /> Requires [dbt v1.6 or higher](/docs/dbt-versions/core) |
| [run](/reference/commands/run) | プロジェクト内のモデルを実行します |   ❌ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [run-operation](/reference/commands/run-operation) | データベースに対して任意のメンテナンスSQLを実行するなど、マクロを呼び出します。 | ❌ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [seed](/reference/commands/seed) | CSVファイルをデータベースにロードします |  ❌ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [show](/reference/commands/show) | 変換後のテーブル行をプレビューします | ✅ |  All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [snapshot](/reference/commands/snapshot) | プロジェクトで定義された「スナップショット」ジョブを実行します |  ❌ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
| [source](/reference/commands/source) | ソースデータを操作するためのツールを提供します（ソースが「最新」であることの検証を含む） | ✅ | All tools<br /> All [supported versions](/docs/dbt-versions/core) |
| [test](/reference/commands/test) | プロジェクトで定義されたテストを実行します  |  ✅ | All tools <br /> All [supported versions](/docs/dbt-versions/core) |
インストールされているdbt Coreまたはdbt Cloud CLIのバージョンを表示するには、[`--version`](/reference/commands/version)フラグを使用してください。(dbt Cloud IDEには適用されません)。すべての[サポート対象バージョン](/docs/dbt-versions/core)で利用できます。
