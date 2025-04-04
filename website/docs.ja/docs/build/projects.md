---
title: "dbt プロジェクトについて"
id: "projects"
pagination_next: null
pagination_prev: null
---

dbt プロジェクトは、プロジェクトのコンテキストとデータの変換方法 (データ セットの構築方法) を dbt に通知します。設計上、dbt は `dbt_project.yml` ファイル、`models` ディレクトリ、`snapshots` ディレクトリなどの dbt プロジェクトの最上位構造を適用します。最上位のディレクトリ内では、組織とデータ パイプラインのニーズを満たす任意の方法でプロジェクトを整理できます。

最低限、プロジェクトに必要なのは `dbt_project.yml` プロジェクト構成ファイルだけです。dbt はさまざまなリソースをサポートしているため、プロジェクトには次のものも含まれる場合があります:

| Resource  | Description  |
| :--- | :--- |
| [models](/docs/build/models) | 各モデルは単一のファイルに存在し、生データを分析可能なデータセットに変換するロジック、または多くの場合、そのような変換の中間ステップとなるロジックが含まれています。 |
| [snapshots](/docs/build/snapshots) | 後で参照できるように、変更可能なテーブルの状態をキャプチャする方法。 |
| [seeds](/docs/build/seeds) | dbt を使用してデータ プラットフォームに読み込むことができる静的データを含む CSV ファイル。 |
| [data tests](/docs/build/data-tests) | プロジェクト内のモデルとリソースをテストするために記述できる SQL クエリ。 |
| [macros](/docs/build/jinja-macros) | 複数回再利用できるコード ブロック。 |
| [docs](/docs/build/documentation) | 構築できるプロジェクトのドキュメント。 |
| [sources](/docs/build/sources) | 抽出およびロード ツールによってウェアハウスにロードされたデータに名前を付け、説明する方法。 |
| [exposures](/docs/build/exposures) | プロジェクトの下流での使用を定義および説明する方法。 |
| [metrics](/docs/build/build-metrics-intro) | プロジェクトのメトリックを定義する方法。 |
| [groups](/docs/build/groups) | グループを使用すると、制限されたコレクション内で共同ノードを編成できます。 |
| [analysis](/docs/build/analyses) | QuickBooks の総勘定元帳など、プロジェクト内の分析 SQL クエリを整理する方法。 |
| [semantic models](/docs/build/semantic-models) | セマンティック モデルは、[MetricFlow](/docs/build/about-metricflow) と [dbt セマンティック レイヤー](/docs/use-dbt-semantic-layer/dbt-sl) の基本的なデータ関係を定義し、セマンティック グラフを使用してメトリックをクエリできるようにします。 |
| [saved queries](/docs/build/saved-queries) | 保存されたクエリは、メトリック、ディメンション、フィルターを dbt DAG に表示されるノードにグループ化することで、再利用可能なクエリを整理します。 |

プロジェクトの構造を構築するときは、組織のワークフローに及ぼす次の影響を考慮する必要があります:

* **ユーザーが dbt コマンドを実行する方法** - パスの選択
* **ユーザーがプロジェクト内をナビゲートする方法** - IDE の開発者として、またはドキュメントの関係者として
* **ユーザーがモデルを構成する方法** - 一括構成の一部はディレクトリ レベルで実行する方が簡単なので、新しいモデルごとに構成ブロックですべてを実行することを覚えておく必要はありません。

## プロジェクト構成
すべての dbt プロジェクトには、`dbt_project.yml` というプロジェクト構成ファイルが含まれています。これは、dbt プロジェクトのディレクトリとその他のプロジェクト構成を定義します。

`dbt_project.yml` を編集して、次のような一般的なプロジェクト構成を設定します:

<div align="center">

| YAML key  | Value description  |
| :--- | :--- |
| [name](/reference/project-configs/name) | プロジェクト名を[スネークケース](https://en.wikipedia.org/wiki/Snake_case)で入力 |
| [version](/reference/project-configs/version) | プロジェクトのバージョン |
| [require-dbt-version](/reference/project-configs/require-dbt-version) | プロジェクトを [dbt Core バージョン](/docs/dbt-versions/core) の範囲でのみ動作するように制限します。 |
| [profile](/reference/project-configs/profile) | dbt がデータ プラットフォームに接続するために使用するプロファイル |
| [model-paths](/reference/project-configs/model-paths) | モデルとソースファイルが存在するディレクトリ |
| [seed-paths](/reference/project-configs/seed-paths) | シードファイルが存在するディレクトリ |
| [test-paths](/reference/project-configs/test-paths) | テストファイルを保存するディレクトリ |
| [analysis-paths](/reference/project-configs/analysis-paths) | 分析結果を保存するディレクトリ |
| [macro-paths](/reference/project-configs/macro-paths) | マクロが存在するディレクトリ |
| [snapshot-paths](/reference/project-configs/snapshot-paths) | スナップショットを保存するディレクトリ |
| [docs-paths](/reference/project-configs/docs-paths) | ドキュメントブロックが保存されるディレクトリ |
| [vars](/docs/build/project-variables) | データのコンパイルに使用するプロジェクト変数 |

</div>

プロジェクト構成の詳細については、[dbt_project.yml](/reference/dbt_project.yml) を参照してください。

## プロジェクトのサブディレクトリ

dbt Cloud のプロジェクト サブディレクトリ オプションを使用すると、dbt がプロジェクトのルート ディレクトリとして使用する git リポジトリ内のサブディレクトリを指定できます。これは、1 つのリポジトリに複数の dbt プロジェクトがある場合や、管理を容易にするために dbt プロジェクト ファイルをサブディレクトリに整理する場合に役立ちます。

dbt Cloud でプロジェクト サブディレクトリ オプションを使用するには、次の手順に従います:

1. ページの右上にある歯車アイコンをクリックし、**アカウント設定** をクリックします。

2. **プロジェクト** で、プロジェクト サブディレクトリとして構成するプロジェクトを選択します。

3. ページの右下隅にある **編集** を選択します。

4. **プロジェクト サブディレクトリ** フィールドに、サブディレクトリの名前を追加します。たとえば、dbt プロジェクト ファイルが `<repository>/finance` というサブディレクトリにある場合は、サブディレクトリとして `finance` と入力します。

    * ネストされたサブディレクトリを参照することもできます。たとえば、dbt プロジェクト ファイルが `<repository>/teams/finance` にある場合は、サブディレクトリとして `teams/finance` と入力します。**注**: プロジェクト サブディレクトリ フィールドでは、先頭または末尾に `/` は必要ありません。

5. 完了したら**保存**をクリックします。

プロジェクト サブディレクトリ オプションを構成すると、dbt Cloud はそれを dbt プロジェクトのルート ディレクトリとして使用します。つまり、`dbt run` や `dbt test` などの dbt コマンドは、指定されたサブディレクトリ内のファイルに対して動作します。プロジェクト サブディレクトリに `dbt_project.yml` ファイルがない場合、dbt プロジェクトを初期化するように求められます。

:::info dbt Cloudプランでのプロジェクトサポート

一部の [プラン](https://www.getdbt.com/pricing) では 1 つの dbt プロジェクトのみがサポートされますが、[エンタープライズ プラン](https://www.getdbt.com/contact) では複数のプロジェクトと dbt Mesh による [プロジェクト間参照](/best-practices/how-we-mesh/mesh-1-intro) が許可されます。

:::

## 新しいプロジェクト

新しいプロジェクトを作成し、GitHub、GitLab、BitBucket などのホストされた Git リポジトリで利用できるようにすることで、他のユーザーと [共有](/docs/collaborate/git-version-control) できます。

データ プラットフォームとの接続を設定したら、[dbt Cloud で新しいプロジェクトを初期化](/guides) して開発を開始できます。または、[コマンド ラインから dbt init](/reference/commands/init) を実行して新しいプロジェクトを設定します。

プロジェクトの初期化中に、dbt はプロジェクト ディレクトリにサンプル モデル ファイルを作成し、すぐに開発を開始できるようにします。

## サンプルプロジェクト

dbt プロジェクトをさらに詳しく調べたい場合は、GitHub で dbt Lab の [Jaffle ショップ](https://github.com/dbt-labs/jaffle_shop) をクローンできます。これは、サンプル構成と役立つメモを含む実行可能なプロジェクトです。

成熟した本番環境プロジェクトがどのようなものかを確認したい場合は、[GitLab データ チームのパブリック リポジトリ](https://gitlab.com/gitlab-data/analytics/-/tree/master/transform/snowflake-dbt) をご覧ください。


## 関連ドキュメント
* [ベスト プラクティス: dbt プロジェクトの構造化方法](/best-practices/how-we-structure/1-guide-overview)
* [dbt Cloud のクイックスタート](/guides)
* [dbt Core のクイックスタート](/guides/manual-install)
