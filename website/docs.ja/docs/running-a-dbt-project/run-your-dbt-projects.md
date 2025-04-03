---
title: "dbt プロジェクトを実行する"
id: "run-your-dbt-projects"
pagination_prev: null
---
dbt プロジェクトは、[dbt Cloud](/docs/cloud/about-cloud/dbt-cloud-features) または [dbt Core](https://github.com/dbt-labs/dbt-core) を使用して実行できます。

- **dbt Cloud**: [dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用して Web ブラウザーから直接開発できるホスト型アプリケーションです。また、コマンド ライン インターフェイス [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) を使用した開発もネイティブでサポートしています。dbt Cloud には、他にも次のような機能があります。

- プロジェクトをより迅速に構築、テスト、実行し、[バージョン管理](/docs/collaborate/git-version-control) するのに役立つ開発環境。
- [dbt プロジェクトのドキュメント](/docs/build/documentation) をチームと共有します。
- dbt Cloud IDE と統合されているため、dbt Cloud UI で開発タスクと環境を実行してシームレスなエクスペリエンスを実現できます。
- dbt Cloud CLI を使用すると、ローカル コマンド ラインから dbt Cloud 開発環境に対して dbt コマンドを開発および実行できます。
- 詳細については、[dbt の開発](/docs/cloud/about-develop-dbt) を参照してください。

- **dbt Core**: [コマンド ライン](/docs/core/installation-overview) から開発できるオープン ソース プロジェクトです。

コマンドラインは、ターミナルや iTerm などのコンピューターのターミナル アプリケーションから使用できます。コマンドラインを使用すると、コンピューターの現在の作業ディレクトリからコマンドを実行したり、その他の作業を行ったりできます。コマンドラインから dbt プロジェクトを実行する前に、dbt プロジェクト ディレクトリで作業していることを確認してください。`cd` (ディレクトリの変更)、`ls` (ディレクトリの内容の一覧表示)、`pwd` (現在の作業ディレクトリ) などのターミナル コマンドを学習すると、システムのディレクトリ構造をナビゲートしやすくなります。

dbt Cloud または dbt Core でよく使用されるコマンドは次のとおりです。

- [dbt run](/reference/commands/run) &mdash; プロジェクトで定義したモデルを実行します
- [dbt build](/reference/commands/build) &mdash; モデル、シード、スナップショット、テストなどの選択したリソースをビルドしてテストします
- [dbt test](/reference/commands/test) &mdash;プロジェクトに定義したテストを実行します

すべての dbt コマンドとその引数 (フラグ) の詳細については、[dbt コマンド リファレンス](/reference/dbt-commands) を参照してください。コマンド ラインからすべての dbt コマンドを一覧表示するには、`dbt --help` を実行します。dbt コマンドの特定の引数を一覧表示するには、`dbt COMMAND_NAME --help` を実行します。

## 関連ドキュメント

- [dbt プロジェクトで作業するためにコンピューターを設定する方法](https://discourse.getdbt.com/t/how-we-set-up-our-computers-for-working-on-dbt-projects/243)
- [モデル選択構文](/reference/node-selection/syntax)
- [dbt Cloud CLI](/docs/cloud/cloud-cli-installation)
- [Cloud IDE 機能](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud#ide-features)
- [dbt は抽出およびロード機能を提供していますか?](/faqs/Project/transformation-tool)
- [dbt コンパイルにデータ プラットフォーム接続が必要な理由](/faqs/Warehouse/db-connection-dbt-compile)
