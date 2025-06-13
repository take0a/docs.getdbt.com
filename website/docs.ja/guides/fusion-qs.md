---
title: "Quickstart for the dbt Fusion engine"
id: "fusion"
# time_to_complete: '30 minutes' commenting out until we test
level: 'Beginner'
icon: 'Guides'
hide_table_of_contents: true
tags: ['dbt Fusion engine', 'dbt Cloud','Quickstart']
recently_updated: true
---

<div style={{maxWidth: '900px'}}>

## 導入

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';

<FusionBeta />

dbt Fusionエンジンは、従来のdbtの考え方に新たなアプローチを加えた強力なエンジンです。Rustで完全に再構築されたFusionにより、dbtプロジェクトのコンパイルと実行がこれまで以上に高速化されます。開発環境や本番環境でFusionを試す前に、実際に動作を確認したいというお客様のニーズにお応えし、このクイックスタートガイドはまさにそれを実現することを目的としています。

### dbt Fusion エンジンについて

Fusion と、このエンジンが提供する強力な機能は、以下の環境でご利用いただけます。

- **dbt Studio:** クラウドベースの dbt Studio IDE をご利用の場合、Fusion の機能が自動的に利用可能となり、インストールは不要です。Fusion エンジンを使用するには、[環境をアップグレード](/docs/dbt-versions/upgrade-dbt-version-in-cloud#dbt-fusion-engine) する必要があります。
- **dbt CLI:** 開発にローカルマシンを使用している場合は、[dbt Fusion エンジン インストール ガイド](/docs/fusion/install-fusion) を参照して、ローカルマシンへのインストール手順をご確認ください。
- **VS Code 拡張機能** [Visual Studio Code (VS Code)](https://code.visualstudio.com/) または [Cursor](https://www.cursor.com/en) IDE をご利用の場合は、[dbt 拡張機能](/docs/install-dbt-extension) をインストールすることで、Fusion の強力な機能の多くをエディターで直接試すことができます。

新機能、変更点、非推奨となった機能について詳しくは、[dbt Fusion エンジンについて](/docs/fusion/about-fusion) をご覧ください。

このガイドでは、dbt 拡張機能と CLI を組み合わせたエクスペリエンスに焦点を当てます。

## 前提条件

このガイドを最大限に活用するには、以下の前提条件を満たす必要があります。
- dbt プロジェクト、Git ワークフロー、およびデータ ウェアハウスの要件に関する基本的な知識が必要です。
- 現在、サポートされているデータ ウェアハウスは Snowflake のみです。今後、さらに多くのアダプタがサポートされる予定です。
- dbt Fusion エンジンを実行するには、macOS (ターミナル) または Windows (PowerShell) マシンが必要です。
- [Visual Studio Code](https://code.visualstudio.com/) がインストールされている必要があります。[Cursor](https://www.cursor.com/en) コード エディタも使用できますが、この手順では VS Code を主に使用します。
    - 問題を回避するため、サードパーティ製の dbt 拡張機能はすべて無効にしてください。

### 学習内容

このクイックスタートガイドでは、dbt Fusion エンジンのインストール方法と使用方法を学習し、迅速かつ効率的にセットアップできるようにします。このガイドでは、以下の手順を説明します。
- 実行可能なプロジェクトを含む、完全に機能する dbt 環境をセットアップする。
- VS Code 経由で dbt 拡張機能と dbt Fusion エンジンをインストールして使用する。
- 環境のターミナルから任意の dbt コマンドを実行する。

質の高い [dbt Learn コースとワークショップ](https://learn.getdbt.com/) で、さらに詳しく学ぶことができます。

### 関連コンテンツ

- [GitHub リポジトリを作成する](/guides/manual-install?step=2)
- [最初のモデルを構築する](/guides/manual-install?step=3)
- [プロジェクトのテストとドキュメント化](/guides/manual-install?step=4)

## インストール

dbt Fusion エンジンと dbt 拡張機能はそれぞれ異なる製品と考えるのが一般的ですが、実際には dbt の潜在能力を最大限に引き出す強力な組み合わせです。Fusion エンジンはまさにその名の通り、エンジンです。dbt 拡張機能と VS Code はシャーシのようなもので、これらを組み合わせることでデータを変換するための強力な手段となります。重要な点として、以下の点にご注意ください。
- dbt Fusion エンジンをインストールし、dbt CLI を使用してスタンドアロンで使用できます。
- Fusion をインストールせずに dbt 拡張機能をインストールして使用することはできません。

[dbt Fusion エンジン](/docs/fusion/install-fusion) および [拡張機能](/docs/install-dbt-extension) のインストールガイドに記載されている重要な手順を以下に示します。

### Fusion macOSおよびLinuxのインストール

ターミナルで次のコマンドを実行します:

```shell
curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update
```

インストール後すぐに `dbtf` を使用するには、新しい `$PATH` が認識されるようにシェルをリロードします:

```shell
exec $SHELL
```

または、ターミナルウィンドウを閉じて再度開くだけです。これにより、更新された環境設定が新しいセッションに読み込まれます。

### Fusion Windows インストール (PowerShell)

PowerShell で次のコマンドを実行します。

```powershell
irm https://public.cdn.getdbt.com/fs/install/install.ps1 | iex
```

インストール後すぐに `dbtf` を使用するには、新しい `Path` が認識されるようにシェルをリロードします:

```powershell
Start-Process powershell
```

または、PowerShell を一度閉じて再度開くだけで、更新された環境設定が新しいセッションに読み込まれます。

### Fusion のインストールを確認する

インストール後、新しいコマンドラインウィンドウを開き、バージョンを確認して Fusion が正しくインストールされていることを確認してください。これらのコマンドは `dbt` を使用して実行できます。また、マシンに別の dbt CLI がインストールされている場合は、Fusion の明確なエイリアスとして `dbtf` を使用することもできます。

```bash
dbtf --version
```

### dbt VS Code 拡張機能をインストールします。

dbt VS Code 拡張機能は、[Visual Studio 拡張機能マーケットプレイス](https://marketplace.visualstudio.com/items?itemName=dbtLabsInc.dbt) から入手できます。VS Code エディターから直接ダウンロードしてください。

1. VS Code（またはカーソル）の [**拡張機能**] タブに移動し、「dbt」を検索します。発行元が `dbt Labs Inc` である拡張機能を見つけます。
    <Lightbox src="/img/docs/extension/extension-marketplace.png" width="60%" title="Search for the extension"/>
2. **インストール** をクリックします。
3. 拡張機能の登録を促すメッセージが表示されます。この手順は今はスキップできますが、[インストール手順](/docs/install-dbt-extension) を確認して後で戻ってください。
4. エディターのステータスバーに **dbt Extension** というラベルが表示されていれば、拡張機能は正常にインストールされています。

## Jaffle Shop プロジェクトを初期化します。

1. コマンドラインから次のコマンドを実行し、サンプルプロジェクトをセットアップしてデータベース接続プロファイルを構成します。

使用する接続プロファイルがまだない場合は、次のコマンドを実行すると、プロンプトが表示され、プロファイルの構成手順が案内されます。

```bash
dbtf init
```

使用したい接続プロファイルが既にある場合は、`--skip-profile-setup` フラグを使用して、生成された `dbt_project.yml` を編集し、`profile: jaffle_shop` を `profile: <YOUR-PROFILE-NAME>` に置き換えます。

```bash
dbtf init --skip-profile-setup
```

対話型プロンプトを使用して新しい認証情報を作成した場合、`init` は最後に自動的に `dbtf debug` を実行します。これにより、新しく作成されたプロファイルがデータベースとの有効な接続を確立しているかどうかがチェックされます。

2. 新しく作成したプロジェクトのディレクトリに移動します:

```bash
cd jaffle_shop
```

3. dbt プロジェクトをビルドします (サンプル データの作成を含む):

```bash
dbtf build
```

以下の作業を行う必要があります:

- 合成データをウェアハウスにロードする
- 開発環境をセットアップして準備する
- プロジェクトをビルドしてテストする

## dbt VS Code 拡張機能を詳しく見る

dbt VS Code 拡張機能は、dbt Fusion エンジンを使用してプロジェクトをコンパイルおよびビルドします。これは、dbt を根本から再構築する強力で超高速なエンジンです。VS Code で Jaffle Shop プロジェクトを使用する方法:

まず、いくつか設定が必要です。

1. **表示** メニューを開き、**コマンドパレット** をクリックして、「ワークスペース: ワークスペースにフォルダーを追加」と入力します。先ほど作成した `jaffle_shop` フォルダを選択します。dbt プロジェクトのルートフォルダをワークスペースに追加しないと、LSP はワークスペース内に読み込まれません。
1. `models/marts/orders.sql` ファイルを開き、`orders` モデルの定義を確認します。これは、以下のすべての例で使用するモデルです。
1. 下部パネルの「Lineage」と「Query Results」、そして右上隅のエディターグループの横にある**dbtアイコン**を見つけます。これらがすべて表示されていれば、拡張機能は正しくインストールされ、実行されています。
    <Lightbox src="/img/docs/extension/extension-running.png" width="60%" title="The VS Code UI with the extension running."/>

これで、これらの素晴らしい機能を実際に試す準備が整いました。

### データとコードをプレビュー

開発プロセスの各ステップで、データ変換に関する貴重な洞察を得ることができます。
モデルの結果と基盤となるデータ構造に、コードから直接アクセスできます。これらのプレビューは、コードをステップごとに検証するのに役立ちます。

1. 右上にある**プレビューファイル**の**テーブルアイコン**を見つけます。クリックすると、**クエリ結果**タブで結果をプレビューできます。
    <Lightbox src="/img/docs/extension/preview-query-results.png" width="60%" title="Preview model query results."/>
1. `orders as (` の上にある **Preview CTE** をクリックして、**Query Results** タブで結果をプレビューします。
    <Lightbox src="/img/docs/extension/preview-cte-query-results-3.png" width="60%" title="Preview CTE query results."/>
1. dbtアイコンとテーブルアイコンの間にある**Compile File**のコードアイコンを探します。これをクリックすると、コンパイル済みのモデルが表示されたウィンドウが開きます。
    <Lightbox src="/img/docs/extension/compile-file-icon.png" width="30%" title="Compile File icon."/>
    <Lightbox src="/img/docs/extension/compile-file.png" width="60%" title="Compile File results."/>

### リネージツールでプロジェクトをナビゲート

データの行き先と同じくらい重要なのは、データがどこにあったかです。拡張機能のリネージツールを使用すると、モデル内のリソースのリネージと列レベルのリネージを視覚化できます。これらの機能により、モデル間の関係性と依存関係をより深く理解できます。

1. **リネージ** タブを開き、このモデルのモデルレベルのリネージを視覚化します。
    <Lightbox src="/img/docs/extension/extension-pane.png" width="60%" title="Visualizing model-level lineage."/>
1. **[表示]** メニューを開き、**[コマンド パレット]** をクリックして「dbt: Show Column Lineage」と入力し、**[系統]** タブに列レベルの系統を表示します。
    <Lightbox src="/img/docs/extension/show-cll.png" width="60%" title="Show column-level lineage."/>

### SQL理解の力を活用しましょう

コーディングは難しくなく、スマートに。オートコンプリートとコンテキストヒントがミスを防ぎ、迅速かつ正確なSQLを記述するのに役立ちます。コミットする前に問題を特定しましょう！

1. **オートコンプリート**の動作を確認するには、`ref('stg_orders')` を削除し、`ref(stg_`と入力し始めると、一致するモデル名のサブセットが表示されます。上下の矢印キーを使って「stg_orders」を選択します。
    <Lightbox src="/img/docs/extension/autocomplete.png" width="60%" title="Autocomplete for a model name."/>
1. いずれかの `*` にマウスを移動すると、選択されている列名とデータ型のリストが表示されます。
    <Lightbox src="/img/docs/extension/hover-star.png" width="60%" title="Hovering over * to see column names and data types."/>

### よく使うdbtコマンドの高速化

テスト、テスト…マイクはオンになっていますか？ オンになっていて、コマンドを超高速で実行する準備が整っています！ コードを様々なdbtコマンドでテストしたい場合は、以下の手順に従ってください。

1. 右上のdbtアイコンをクリックすると、拡張機能固有のコマンドのリストが表示されます:
    <Lightbox src="/img/docs/extension/run-command.png" width="60%" title="Select a command via the dbt icon."/>
1. **[表示]** メニューを開き、**[コマンド パレット]** をクリックして、コマンド バーに `>dbt:` と入力すると、使用可能な新しいコマンドがすべて表示されます。
    <Lightbox src="/img/docs/extension/extension-commands-all.png" width="60%" title="dbt commands in the command bar."/>

いくつか選んで、どんな機能があるか試してみてください😎

これはほんの始まりに過ぎません。今後さらに多くの機能が利用可能になり、さらに多くの機能が追加予定です。dbt Fusion エンジンと dbt VS Code 拡張機能に関するすべての情報は、当社のリソースをご覧ください。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />

## トラブルシューティング

#### `dbt 言語サーバーがこのワークスペースで実行されていません` エラーへの対処

`dbt language server is not running in this workspace` エラーを解決するには、dbt プロジェクトフォルダをワークスペースに追加する必要があります。

1. VS Code で、ツールバーの **ファイル** をクリックし、**ワークスペースにフォルダーを追加** を選択します。
2. ワークスペースに追加する dbt プロジェクトファイルを選択します。
3. ワークスペースを保存するには、**ファイル** をクリックし、**ワークスペースに名前を付けて保存** を選択します。
4. ワークスペースを保存する場所に移動します。

これでエラーが解決し、dbt プロジェクトが属するワークスペースが開きます。ワークスペースの詳細については、[VS Code ワークスペースとは](https://code.visualstudio.com/docs/editing/workspaces/workspaces) を参照してください。

</div>
