---
title: DuckDB を使用した dbt Core のクイックスタート
id: duckdb
description: "Learn to use dbt Core using DuckDB."
hoverSnippet: "Learn to use dbt Core using DuckDB."
platform: 'dbt-core'
icon: 'duckdb-seeklogo'
level: 'Beginner'
hide_table_of_contents: true
tags: ['dbt Core','Quickstart']
---

<div style={{maxWidth: '900px'}}>

## はじめに

このクイックスタート ガイドでは、dbt Core を DuckDB とともに使用して、迅速かつ効率的にセットアップする方法を学びます。[DuckDB](https://duckdb.org/) は、分析ワークロード向けに設計されたオープンソースのデータベース管理システムです。大規模なデータセットに迅速かつ簡単にアクセスできるように設計されているため、データ分析タスクに最適です。


このガイドでは、次の方法を説明します。

- dbt Labs が提供するテンプレートを使用して、[仮想開発環境を作成する](/docs/core/pip-install#using-virtual-environments)。
- 運用可能で実行可能なプロジェクトを使用して、完全に機能する dbt 環境をセットアップします。コードスペースは DuckDB データベースに自動的に接続し、米国の複数の都市で食品や飲料を販売している架空のカフェ Jaffle Shop から 1 年分のデータを読み込みます。
- `jaffle_shop_duck_db` リポジトリで説明されている手順を実行しますが、基礎となるコードをさらに詳しく調べる場合は、Jaffle Shop テンプレートの [README](https://github.com/dbt-labs/jaffle_shop_duckdb/blob/duckdb/README.md) を参照してください。
- 環境のターミナルから任意の dbt コマンドを実行します。
- Jaffle Shop カフェ用に、より大きなデータセットを生成します (たとえば、1 年分ではなく 5 年分のデータ)。

高品質の [dbt Learn コースとワークショップ](https://learn.getdbt.com) を通じて、さらに詳しく学ぶことができます。


### 関連コンテンツ


- [DuckDB のセットアップ](/docs/core/connect-data-platform/duckdb-setup)
- [GitHub リポジトリの作成](/guides/manual-install?step=2)
- [最初のモデルの構築](/guides/manual-install?step=3)
- [プロジェクトのテストとドキュメント化](/guides/manual-install?step=4)


## 前提条件

- DuckDB を dbt Core で使用する場合は、dbt コマンドライン インターフェース (CLI) を使用する必要があります。現在、DuckDB は dbt Cloud ではサポートされていません。
- ターミナルの基本を理解しておくことが重要です。特に、コンピューターのディレクトリ構造を簡単にナビゲートするには、`cd`、`ls`、`pwd` を理解しておく必要があります。
- [GitHub アカウント](https://github.com/join) を持っていること。

## dbt Core 用に DuckDB を設定する

このセクションでは、ローカル (Mac および Windows) 環境と Web ブラウザーで使用するために DuckDB を設定するための手順を順を追って説明します。

リポジトリには、dbt Core、DuckDB、およびその他の必要な依存関係をインストールするために使用される [`requirements.txt`](https://github.com/dbt-labs/jaffle_shop_duckdb/blob/duckdb/requirements.txt) ファイルがあります。このファイルを確認すると、マシンにインストールされる内容を確認できます。通常、このファイルは、`dbt_project.yml` などの他の重要なファイルとともに、プロジェクトのルート ディレクトリにあります。それ以外の場合は、後の手順でその方法を説明します。

以下は、`dbt_project.yml` などの他の重要なファイルとともに、`requirements.txt` ファイルの例です:


```shell

/my_dbt_project/
├── dbt_project.yml
├── models/
│   ├── my_model.sql
├── tests/
│   ├── my_test.sql
└── requirements.txt

```

詳細については、[DuckDB セットアップ](/docs/core/connect-data-platform/duckdb-setup)を参照してください。

<Tabs>
  <TabItem value="local" label="Local">


1. まず、ターミナルで次のコマンドを実行して、Jaffle Shop の git リポジトリを [クローン](https://git-scm.com/docs/git-clone) します:



    ```bash
    git clone https://github.com/dbt-labs/jaffle_shop_duckdb.git

    ```

2. コマンドラインから docs-duckdb ディレクトリに移動します:

    ```shell

    cd jaffle_shop_duck_db

    ```


3. 仮想環境に dbt Core と DuckDB をインストールします。

    <Expandable alt_header="Example for Mac" >

    ```shell

    python3 -m venv venv
    source venv/bin/activate
    python3 -m pip install --upgrade pip
    python3 -m pip install -r requirements.txt
    source venv/bin/activate

    ```
    </Expandable>

    <Expandable alt_header="Example for Windows" >

    ```shell

    python -m venv venv
    venv\Scripts\activate.bat
    python -m pip install --upgrade pip
    python -m pip install -r requirements.txt
    venv\Scripts\activate.bat

    ```

    </Expandable>

    <Expandable alt_header="Example for Windows PowerShell" >

    ```shell

    python -m venv venv
    venv\Scripts\Activate.ps1
    python -m pip install --upgrade pip
    python -m pip install -r requirements.txt
    venv\Scripts\Activate.ps1

    ```
    </Expandable>


4. 次の [dbt コマンド](/reference/dbt-commands) を実行して、コマンド ラインからプロファイルが正しく設定されていることを確認します。


    - [dbt compile](/reference/commands/compile) &mdash; プロジェクトのソースファイルから実行可能なSQLを生成します
    - [dbt run](https://docs.getdbt.com/reference/commands/run) &mdash; プロジェクトをコンパイルして実行する
    - [dbt test](https://docs.getdbt.com/reference/commands/test) &mdash; プロジェクトをコンパイルしてテストします
    - [dbt build](https://docs.getdbt.com/reference/commands/build) &mdash; プロジェクトをコンパイル、実行、テストします
    - [dbt docs generate](/reference/commands/cmd-docs#dbt-docs-generate) &mdash; プロジェクトのドキュメントを生成します。
    - [dbt docs serve](/reference/commands/cmd-docs#dbt-docs-serve) &mdash; ポート 8080 で Web サーバーを起動してドキュメントをローカルで提供し、デフォルトのブラウザーでドキュメント サイトを開きます。

詳細については、[dbt コマンド リファレンス](/reference/dbt-commands) を参照してください。

成功した場合の出力は次のようになります:

```jinja

(venv) ➜  jaffle_shop_duckdb git:(duckdb) dbt build
15:10:12  Running with dbt=1.8.1
15:10:13  Registered adapter: duckdb=1.8.1
15:10:13  Found 5 models, 3 seeds, 20 data tests, 416 macros
15:10:13  
15:10:14  Concurrency: 24 threads (target='dev')
15:10:14  
15:10:14  1 of 28 START seed file main.raw_customers ..................................... [RUN]
15:10:14  2 of 28 START seed file main.raw_orders ........................................ [RUN]
15:10:14  3 of 28 START seed file main.raw_payments ...................................... [RUN]
....

15:10:15  27 of 28 PASS relationships_orders_customer_id__customer_id__ref_customers_ .... [PASS in 0.32s]
15:10:15  
15:10:15  Finished running 3 seeds, 3 view models, 20 data tests, 2 table models in 0 hours 0 minutes and 1.52 seconds (1.52s).
15:10:15  
15:10:15  Completed successfully
15:10:15  
15:10:15  Done. PASS=28 WARN=0 ERROR=0 SKIP=0 TOTAL=28

```
データをクエリするには、コマンドラインから実行できる便利なコマンドがいくつかあります:

- `dbt show --select "raw_orders"` - データ ウェアハウスに対してクエリを実行し、ターミナルで結果をプレビューします。
- [`dbt source`](/reference/commands/source) - ソース データの操作時に便利な [`dbt source freshness`](/reference/commands/source#dbt-source-freshness) などのサブコマンドを提供します。
    - `dbt source freshness` - 特定のソース テーブルの鮮度 (最新度) を確認します。

:::note

このプロジェクトをデータ ウェアハウス (この DuckDB デモ以外) で実行することにした場合、手順は失敗します。ウェアハウス用にプロジェクト ファイルを再構成する必要があります。コミュニティ提供のアダプターを使用している場合は、必ずこれを考慮してください。

:::


### Troubleshoot

    <Expandable alt_header="Could not set lock on file error" >

    ```Jinja

    IO Error: Could not set lock on file "jaffle_shop.duckdb": Resource temporarily unavailable

    ```

    これは DuckDB の既知の問題です。データベースをロックしているセッションから切断してみてください。DBeaver を使用している場合は、DBeaver をシャットダウンする必要があります (切断が常に機能するとは限りません)。

    最後の手段として、データベース ファイルを削除すると、再び動作できるようになります (ただし、すべてのデータが失われます)。

    </Expandable>


  </TabItem>
 
  <TabItem value="web" label="Web browser">

1. GitHub アカウントにログインした後、`jaffle-shop-template` [リポジトリ](https://github.com/dbt-labs/jaffle_shop_duckdb) に移動します。
1. ページ上部の **Use this template** をクリックし、**Create new repository** を選択します。
1. 新しいリポジトリのオプションの設定が完了したら、**Create repository from template** をクリックします。
1. **Code** をクリックします (新しいリポジトリのページ上部)。**Codespaces** タブで、**Create codespace on main** を選択します。コンピューターの設定に応じて、VSCode が実行されているコードスペース開発環境の新しいブラウザー タブが開くか、コードスペースを含む新しい VSCode ウィンドウが開きます。
1. `postCreateCommand` コマンドが完了するまで待機して、コードスペースのビルドが完了するまで待機します。これには数分かかる場合があります。

    <Lightbox src="/img/codespace-quickstart/postCreateCommand.png" title="Wait for postCreateCommand to complete" />

    このコマンドが完了すると、コードスペース開発環境の使用を開始できます。コマンドを実行したターミナルは閉じられ、新しいターミナルにプロンプ​​トが表示されます。

1. ターミナルのプロンプトで、任意の dbt コマンドを実行できます。例:

    ```shell
    /workspaces/test (main) $ dbt build
    ```

    また、[duckcli](https://duckdb.org/docs/api/cli/overview.html) を使用して、コマンドラインからウェアハウスに対して SQL を記述したり、`reports` ディレクトリに用意されている [Evidence](https://evidence.dev/) プロジェクトでレポートを作成したりすることもできます。
    
    詳細については、[dbt コマンド リファレンス](https://docs.getdbt.com/reference/dbt-commands) を参照してください。一般的なコマンドは次のとおりです。
    
    - [dbt compile](/reference/commands/compile) &mdash; プロジェクトのソースファイルから実行可能なSQLを生成します
    - [dbt run](https://docs.getdbt.com/reference/commands/run) &mdash; プロジェクトをコンパイルして実行する
    - [dbt test](https://docs.getdbt.com/reference/commands/test) &mdash; プロジェクトをコンパイルしてテストします
    - [dbt build](https://docs.getdbt.com/reference/commands/build) &mdash; プロジェクトをコンパイル、実行、テストします


  </TabItem>

</Tabs>







## より大きなデータセットを生成する

より大規模な Jaffle Shop データを扱いたい場合は、コードスペース内から任意の年数の架空のデータを生成できます。

1. [jafgen](https://pypi.org/project/jafgen/) という Python パッケージをインストールします。ターミナルのプロンプトで、次のコマンドを実行します。

    ```shell
    python -m pip install jafgen
    ```

1. インストールが完了したら、次を実行します:
    ```shell
    jafgen [number of years to generate] # e.g. jafgen 6
    ``` 
    `NUMBER_OF_YEARS` を、シミュレートする年数に置き換えます。たとえば、6 年間のデータを生成するには、`jafgen --years 6` を実行します。このコマンドは、CSV ファイルを構築して `jaffle-data` フォルダーに保存し、`sources.yml` ファイルと [dbt-duckdb](/docs/core/connect-data-platform/duckdb-setup) アダプターに基づいて自動的にソース化されます。

年数を増やすと、Jaffle Shop の店舗数と規模が拡大するため、データの生成にかかる時間は飛躍的に増加します。データ サイズと構築時間のバランスを適切に保つために、dbt Labs では最大 6 年を推奨しています。
## 次のステップ

dbt Core、DuckDB、Jaffle Shop データが稼働しているので、dbt の機能を調べることができます。dbt プロジェクトとコマンドについて理解を深めるには、次の資料を参照してください。

- [プロジェクトについて](/docs/build/projects) ページでは、dbt プロジェクトの構造とそのコンポーネントについて説明します。
- [dbt コマンド リファレンス](/reference/dbt-commands) では、使用可能なさまざまなコマンドとその機能について説明します。
- [dbt Labs コース](https://courses.getdbt.com/collections) では、dbt エキスパートになるために役立つように設計された、さまざまな初級、中級、上級の学習モジュールを提供しています。
- dbt の可能性と組織で何ができるかがわかったら、[dbt Cloud](https://www.getdbt.com/signup) の無料トライアルにサインアップしてください。これは、今日 dbt を展開する最も速くて簡単な方法です。
- 既存のデータ ウェアハウスへの統合を開始するには、他の [クイックスタート ガイド](/guides?tags=Quickstart) を確認してください。

さらに、DuckDB の使用の基本を新たに理解したら、[プロジェクトを文書化](/guides/duckdb#document-your-project)、[変更をコミット](/guides/duckdb#commit-your-changes)、[ジョブをスケジュール](/guides/duckdb#schedule-a-job)してセットアップを最適化することを検討してください。

### プロジェクトを文書化する

DuckDB を使用して dbt プロジェクトをドキュメント化するには、次の手順に従います。

- `dbt docs generate` コマンドを使用して、dbt プロジェクトとウェアハウスに関する情報を `manifest.json` ファイルと `catalog.json` ファイルにコンパイルします。
- [`dbt docs serve`](/reference/commands/cmd-docs#dbt-docs-serve) コマンドを実行して、生成された `.json` ファイルを使用してローカル Web サイトを作成します。これにより、プロジェクトのドキュメントを Web ブラウザーで表示できます。
- YAML ファイルの `description` キーを使用して、モデル、列、ソースに [descriptions](/reference/resource-properties/description) を追加して、ドキュメントを強化します。

### 変更をコミットする

変更をコミットして、リポジトリが最新のコードで更新されていることを確認します。

1. プロジェクト用に作成した GitHub リポジトリで、ターミナルで次のコマンドを実行します:

```shell
git add 
git commit -m "Your commit message"
git push
```

2. GitHub リポジトリに戻り、新しいファイルが追加されたことを確認します。

### ジョブをスケジュールする

1. dbt Core がインストールされ、DuckDB インスタンスに接続するように構成されていることを確認します。
2. dbt プロジェクトを作成し、[`models`](/docs/build/models)、[`seeds`](/reference/seed-properties)、[`tests`](/reference/commands/test) を定義します。
3. [Prefect](/docs/deploy/deployment-tools#prefect) などのスケジューラを使用して、dbt の実行をスケジュールします。指定した間隔で dbt コマンドをトリガーする DAG (有向非巡回グラフ) を作成できます。
4. [`dbt run`](/reference/commands/run)、`dbt test` などの dbt コマンドを実行するスクリプトを作成します。
5. 選択したスケジューラを使用して、必要な頻度でスクリプトを実行します。

<ConfettiTrigger>

Cガイドを最後までお読みいただき、ありがとうございます 🎉!

</ConfettiTrigger>

</div>



