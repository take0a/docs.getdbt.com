---
title: "dbt Core とインストールについて"
description: "Install dbt Core locally to begin transforming your data."
pagination_next: "docs/core/pip-install"
pagination_prev: null
---

[dbt Core](https://github.com/dbt-labs/dbt-core) は、コマンドラインから開発して dbt プロジェクトを実行できるオープンソース プロジェクトです。

dbt Core を使用する場合、ワークフローは通常次のようになります。

1. **コード エディターで dbt プロジェクトをビルドします。** 一般的な選択肢としては、VSCode や Atom などがあります。

2. **コマンド ラインからプロジェクトを実行します。** macOS にはデフォルトのターミナル プログラムが付属していますが、コード エディター内で iTerm またはコマンド ライン プロンプトを使用して dbt コマンドを実行することもできます。

:::info dbt プロジェクトで作業するためにコンピューターを設定する方法

dbt Core を使用して dbt プロジェクトを実行する場合に推奨されるセットアップについては、[ガイド](https://discourse.getdbt.com/t/how-we-set-up-our-computers-for-working-on-dbt-projects/243) を作成しました。

:::

コマンドラインを使用している場合は、より効率的に作業するために、ターミナルの基本を学ぶことをお勧めします。特に、コンピューターのディレクトリ構造を簡単にナビゲートできるようにするために、`cd`、`ls`、`pwd` を理解することが重要です。

## dbt Core をインストールする

次のいずれかの方法を使用して、コマンド ラインで dbt Core をインストールできます。

- [pip を使用して dbt をインストールする](/docs/core/pip-install) (推奨)
- [Docker イメージを使用して dbt をインストールする](/docs/core/docker-install)
- [ソースから dbt をインストールする](/docs/core/source-install)
- [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) を使用してローカルで開発することもできます。dbt Cloud CLI と dbt Core はどちらも、dbt コマンドを実行できるコマンド ライン ツールです。主な違いは、dbt Cloud CLI は dbt Cloud のインフラストラクチャに合わせて調整されており、そのすべての [機能](/docs/cloud/about-cloud/dbt-cloud-features) と統合されていることです。

## dbt Core のアップグレード

dbt は、dbt プロジェクトのアップグレード中に [一般的なベスト プラクティス](/blog/upgrade-dbt-without-fear) を理解するためのリソースを多数提供しているほか、各 [マイナー リリースとメジャー リリース](/docs/dbt-versions/core) に必要な変更を強調した詳細な [移行ガイド](/docs/dbt-versions/core) も提供しています。

- [`pip` のアップグレード](/docs/core/pip-install#change-dbt-core-versions)

## dbt データ プラットフォームとアダプタについて

dbt は、さまざまなデータ プラットフォーム (データベース、クエリ エンジン、その他の SQL 対応テクノロジ) で動作します。これは、それぞれ専用の _アダプタ_ を使用して行われます。dbt Core をインストールするときは、データベースに固有のアダプタもインストールする必要があります。詳細については、[サポートされているデータ プラットフォーム](/docs/supported-data-platforms) を参照してください。

:::tip Pro tip: --helpフラグの使用

dbt を含むほとんどのコマンドライン ツールには、使用可能なコマンドと引数を表示するために使用できる `--help` フラグがあります。たとえば、dbt で `--help` フラグを使用するには、次の 2 つの方法があります。<br /><br />
&mdash; `dbt --help`: dbt で使用可能なコマンドを一覧表示します<br />
&mdash; `dbt run --help`: `run` コマンドで使用可能なフラグを一覧表示します

:::


