---
title: "About the Studio IDE"
id: develop-in-the-cloud
description: "Develop, test, run, and build in the Cloud IDE. You can compile dbt code into SQL and run it against your database directly"
sidebar_label: About the IDE
tags: [IDE]
pagination_next: "docs/cloud/dbt-cloud-ide/ide-user-interface"
pagination_prev: null
---

<Constant name="cloud" /> 統合開発環境 (<Constant name="cloud_ide" />) は、dbt プロジェクトの構築、テスト、実行、バージョン管理を行うための単一の Web ベース インターフェースです。dbt コードを SQL にコンパイルし、データベース上で直接実行します。

<Constant name="cloud_ide" /> には、開発とガバナンスをより迅速かつ効率的に行うための [キーボード ショートカット](/docs/cloud/dbt-cloud-ide/keyboard-shortcuts) と [編集機能](/docs/cloud/dbt-cloud-ide/ide-user-interface#editing-features) がいくつか用意されています。

- SQL の構文ハイライト - コードのさまざまな部分を簡単に区別できるため、構文エラーが減り、読みやすさが向上します。
- AI コパイロット - AI 搭載アシスタントの [<Constant name="copilot" />](/docs/cloud/dbt-copilot) を使用すると、自然言語を使用して [コードを生成](/docs/cloud/dbt-cloud-ide/develop-copilot#generate-and-edit-code) したり、[リソースを生成](/docs/cloud/dbt-cloud-ide/develop-copilot#generate-resources) (ドキュメント、テスト、セマンティック モデルなど) をボタンをクリックするだけで生成できます。詳細については、[<Constant name="copilot" /> を使用した開発](/docs/cloud/dbt-cloud-ide/develop-copilot) をご覧ください。
- オートコンプリート - 入力時にテーブル名、引数、列名の候補が表示されるため、時間の節約になり、入力ミスも減ります。
- コードの [フォーマットと lint](/docs/cloud/dbt-cloud-ide/lint-format) &mdash; SQL コードを簡単に標準化および修正できます。
- ナビゲーション ツール &mdash; コード内を簡単に移動したり、特定の行にジャンプしたり、テキストを検索して置換したり、プロジェクト ファイル間を移動したりできます。
- バージョン管理 &mdash; 数回のクリックでコードのバージョンを管理できます。
- プロジェクト ドキュメント &mdash; dbt プロジェクトの [プロジェクト ドキュメント](#build-and-document-your-projects) をリアルタイムで生成して表示できます。
- ビルド、テスト、実行ボタン &mdash; ボタンをクリックするか、<Constant name="cloud_ide" /> コマンド バーを使用して、プロジェクトをビルド、テスト、実行できます。

これらの [機能](#dbt-cloud-ide-features) により、経験豊富な開発者と初心者の開発者の両方に適した、効率的な SQL コーディングのための強力な編集環境が実現します。

<DocCarousel slidesPerView={1}>

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-basic-layout.jpg" width="85%" title="The Studio IDE includes version control,files/folders, an editor, a command/console, and more."/>

<Lightbox src src="/img/docs/dbt-cloud/cloud-ide/cloud-ide-v2.jpg" width="85%" title="Enable dark mode for a great viewing experience in low-light environments."/>
</DocCarousel>

:::tip Disable ad blockers

<Constant name="cloud" /> のご利用体験を向上させるため、広告ブロッカーをオフにすることをお勧めします。一部のプロジェクトファイル名（例: `google_adwords.sql`）は広告トラフィックに似ており、広告ブロッカーのトリガーとなる可能性があるためです。

:::

## 前提条件

- [<Constant name="cloud" /> アカウント](https://www.getdbt.com/signup) と [Developer シートライセンス](/docs/cloud/manage-access/seats-and-users)
- Git リポジトリがセットアップされており、Git プロバイダーに `write` アクセス権限が有効になっている必要があります。詳細なセットアップ手順については、[GitHub アカウントの接続](/docs/cloud/git/connect-github) または [Git URL によるプロジェクトのインポート](/docs/cloud/git/import-a-project-by-git-url) をご覧ください。
- dbt プロジェクトが [データ プラットフォーム](/docs/cloud/connect-data-platform/about-connections) に接続されていること
- [開発環境と開発認証情報](#get-started-with-the-cloud-ide) がセットアップされていること
- 環境は dbt バージョン 1.0 以上である必要があります

## Studio IDE の機能

<Constant name="cloud_ide" /> には、データモデルの開発、ビルド、コンパイル、実行、テストを容易にする機能が備わっています。

<Constant name="cloud_ide" /> とそのユーザーインターフェース要素の操作方法については、[<Constant name="cloud_ide" /> ユーザーインターフェース](/docs/cloud/dbt-cloud-ide/ide-user-interface) ページを参照してください。

| Feature  |  Description |
|---|---|
| [**<Constant name="cloud_ide" /> ショートカット**](/docs/cloud/dbt-cloud-ide/keyboard-shortcuts) | 適切なキーボードショートカットを選択すると、<Constant name="cloud_ide" /> 内のさまざまな[コマンドとアクション](/docs/cloud/dbt-cloud-ide/keyboard-shortcuts)にアクセスできます。変更したモデルのビルドや、前回の失敗からのビルドの再開など、よく使用するタスクにはショートカットを使用してください。 |
| **IDE バージョン管理** | <Constant name="cloud_ide" /> バージョン管理セクションと git ボタンを使用すると、[バージョン管理](/docs/cloud/git/version-control-basics) の概念を <Constant name="cloud_ide" /> のプロジェクトに直接適用できます。 <br /><br /> - ブランチを作成または変更し、git ボタンを使用して git コマンドを実行します。<br /> - 編集したファイルを右クリックして、個々のファイルをコミットまたは元に戻します。<br /> - [マージの競合を解決](/docs/cloud/git/merge-conflicts)<br /> - ブランチ名をクリックして、リポジトリに直接リンクします。<br /> - 保護されたプライマリ ブランチでファイルを編集、フォーマット、または lint し、dbt コマンドを実行して、新しいブランチにコミットします。<br /> - プル リクエストを作成する前に、Git の diff ビューを使用してファイルの変更内容を確認します。<br /> - **ブランチのプルーニング** [ボタン](/docs/cloud/dbt-cloud-ide/ide-user-interface#prune-branches-modal) を使用して、リモート リポジトリから削除されたローカル ブランチを削除し、ブランチ管理を整理します。<br /> - [gitコミット](/docs/cloud/dbt-cloud-ide/git-commit-signing) を実行して、それらを '検証済み' としてマークします。<Lifecycle status="managed,managed_plus" /> |
| **プレビューとコンパイル ボタン** | 編集して保存した後、コード、dbt コードのスニペット、または dbt モデルのいずれかを [コンパイルまたはプレビュー](/docs/cloud/dbt-cloud-ide/ide-user-interface#console-section) できます。 |
| [**<Constant name="copilot" />**](/docs/cloud/dbt-cloud-ide/develop-copilot)| 自然言語を使用して [コードを生成](/docs/cloud/dbt-cloud-ide/develop-copilot#generate-and-edit-code)し、[リソースを生成](/docs/cloud/dbt-cloud-ide/develop-copilot#generate-resources) (ドキュメント、テスト、メトリック、セマンティック モデルなど) をボタンをクリックするだけで実行できる、強力な AI 搭載アシスタントです。<Lifecycle status="self_service,managed,managed_plus" />。|
| **ビルド、テスト、実行ボタン** | ボタンをクリックするか、コマンド バーを使用して、プロジェクトをビルド、テスト、実行します。 |
| **コマンド バー** | <Constant name="cloud_ide" /> の下部にあるコマンド バーからコマンドを入力して実行できます。[豊富なモデル選択構文](/reference/node-selection/syntax)を使用して、<Constant name="cloud" /> 内で直接 [dbt コマンド](/reference/dbt-commands) を実行します。また、バーの左側にある [履歴] をクリックすると、以前の実行の履歴、ステータス、ログを表示することもできます。|
| **ドラッグアンドドロップ** | ファイルエクスプローラーにあるファイルをドラッグアンドドロップし、<Constant name="cloud_ide" /> 上部のファイルパンくずリストを使って素早く直線的に移動できます。パンくずリストファイルを右クリックすると、同じファイル内の隣接するファイルへアクセスできます。 |
| **タブとファイルの整理** | - タブを移動して、IDE での作業を整理します <br /> - タブを右クリックすると、重複ファイルなどのアクションのリストが表示され、選択できます <br /> - 複数の未保存のタブを閉じて、作業を一括保存します <br /> - ファイルをダブルクリックして、ファイルの名前を変更します |
| **検索と置換** | - Command-F または Control-F を押すと、IDE の現在のファイルの右上隅に検索と置換バーが開きます。IDE は、現在のファイルとコード アウトラインで検索結果をハイライト表示します。<br /> - 複数の一致がある場合は、上矢印と下矢印を使用して、現在のファイルでハイライト表示された一致を確認できます。<br /> - 左矢印を使用して、テキストを別のものに置き換えます。 |
| **複数選択** | 小規模な編集や同時編集のために、複数の選択範囲を作成できます。以下のコマンドは、カーソルを追加し、簡単に上下にカーソルを挿入するための一般的な方法です。<br /><br /> - Option + Command + 下矢印キーまたは Ctrl + Alt + 下矢印キー<br /> - Option + Command + 上矢印キーまたは Ctrl + Alt + 上矢印キー<br /> - Option キーを押しながら領域をクリック、または Ctrl + Alt キーを押しながら領域をクリック<br /> |
| **Lint とフォーマット** | SQLFluff、sqlfmt、Prettier、Black を活用し、ボタンをクリックするだけでファイルを [Lint とフォーマット](/docs/cloud/dbt-cloud-ide/lint-format) できます。|
| **dbt オートコンプリート** | 開発を高速化するオートコンプリート機能:<br /><br /> - `ref` を使用してモデル名をオートコンプリートします<br /> - `source` を使用してソース名 + テーブル名をオートコンプリートします<br /> - `macro` を使用して引数をオートコンプリートします<br /> - `env var` を使用して環境変数をオートコンプリートします<br /> - ハイフン (-) の入力を開始すると、YAML ファイルでインライン オートコンプリートが使用されますbr /> - ボタンをクリックするだけで、dbt ソースからモデルが自動的に作成されます。 |
| **IDE の <Term id="dag" />** | 左から右へと、モデルが構成要素としてどのように使用され、生のソースデータがクリーンアップされたモジュール化された派生ピースに変換され、DAG の右端に最終的な出力が表示されるかを確認できます。デフォルトの表示は 2+model+2（デフォルトでは 2 ノード先まで表示されます）ですが、+model+（完全な <Term id="dag" />）に変更できます。`--exclude` フラグはサポートされていないことに注意してください。 |
| **ステータスバー** | この領域には、<Constant name="cloud_ide" /> とプロジェクトのステータスに関する有用な情報が表示されます。また、ライトモードまたはダークモードの有効化、<Constant name="cloud_ide" /> の再起動、[リポジトリの再クローン](/docs/cloud/git/version-control-basics)などの追加オプションも利用できます。|
| **ダーク モード** | Cloud <Constant name="cloud_ide" /> のステータス バーからダーク モードを有効にすると、暗い環境でも優れた表示エクスペリエンスが得られます。 |


### コード生成

<Constant name="cloud_ide" /> には、**CodeGenCodeLens** という強力な機能が付属しています。この機能を使うと、ボタンをクリックするだけでソースからモデルを簡単に作成できます。この機能を使用するには、ソース YAML ファイル内の各テーブルの横にある「モデルを生成」アクションをクリックします。これにより、基本的なステージングモデルが自動的に作成され、それを拡張することができます。この機能は、モデル生成の最初のステップを自動化することで、ワークフローを効率化します。

### dbt YAML 検証

dbt-jsonschema を使用して dbt YAML ファイルを検証することで、<Constant name="cloud_ide" /> のオートコンプリート機能やアシスタンス機能を活用できます。また、YAML ファイルの構造と構文に関するフィードバックが即座に提供されるため、プロジェクト構成が必要な基準を満たしていることを確認できます。

## Cloud IDE を使い始めましょう

Cloud <Constant name="cloud_ide" /> の優れた機能を体験するには、まず [<Constant name="cloud" /> 開発環境](/docs/dbt-cloud-environments) をセットアップする必要があります。以下の手順では、開発者認証情報を設定し、<Constant name="cloud_ide" /> にアクセスする方法について説明します。新しいプロジェクトを作成する場合は、プロジェクトのセットアップ時に自動的に設定されます。

<Constant name="cloud_ide" /> は、データプラットフォームに接続するために開発者認証情報を使用します。これらの開発者認証情報は、ユーザー固有のものである必要があり、スーパーユーザーの認証情報や、dbt の本番環境展開で使用する認証情報と同じものは使用しないでください。

開発者認証情報を設定します。

1. **Your Profile** 設定の **Credentials** に移動します。これは `https://YOUR_ACCESS_URL/settings/profile#credentials` でアクセスできます。`YOUR_ACCESS_URL` は、リージョンとプランの [適切なアクセス URL](/docs/cloud/about-cloud/access-regions-ip-addresses) に置き換えてください。
2. リストから該当するプロジェクトを選択します。
3. ページの右下にある **Edit** をクリックします。
4. **Development Credentials** に詳細を入力します。
5. **Save** をクリックします。

<Lightbox src="/img/docs/dbt-cloud/refresh-ide/dev-credentials.jpg" width="85%" height="100" title="Configure developer credentials in your Profile"/>

6. ページ上部の **Develop** をクリックして、<Constant name="cloud_ide" /> にアクセスします。
7. プロジェクトを初期化し、<Constant name="cloud_ide" /> とその便利な [機能](#cloud-ide-features) に慣れましょう。

これで、モデルの開発と構築を始める準備が整いました。 🎉!  

### 考慮事項

- <Constant name="cloud" /> のエクスペリエンスを向上させるため、広告ブロッカーを無効にすることをおすすめします。`google_adwords.sql` などの一部のプロジェクトファイル名は広告トラフィックと類似しており、広告ブロッカーが起動する可能性があるためです。
- パフォーマンスを維持するため、6 GB を超えるリポジトリにはファイルサイズ制限があります。6 GB を超えるリポジトリをお持ちの場合は、<Constant name="cloud" /> を実行する前に [dbt サポート](mailto:support@getdbt.com) にお問い合わせください。
- <Constant name="cloud_ide" /> のアイドルセッションタイムアウトは 1 時間です。
- <Expandable alt_header="起動プロセスと作業の保持について">
  
    以下のセクションでは、<Constant name="cloud_ide" /> の起動プロセスと作業データの保持について説明します。

    - #### 起動プロセス
      <Constant name="cloud_ide" /> を使用または起動する際の起動状態は 3 つあります。
      - **作成開始 &mdash;** これは、IDE を初めて起動する状態です。これは *コールドスタート* とも呼ばれます (下記参照)。Git リポジトリのクローンが作成されているため、この状態はより時間がかかることが予想されます。
      - **コールドスタート &mdash;** これは、新しい開発セッションを開始するプロセスです。このセッションは 1 時間利用できます。環境は、最後のアクティビティから 1 時間後に自動的にオフになります。これにはコンパイル、プレビュー、または dbt の呼び出しが含まれますが、ファイルの編集と保存は含まれません。
      - **ホットスタート &mdash;** これは、最後のアクティビティから 1 時間以内に既存またはアクティブな開発セッションを再開する状態です。 <br /><br />

    - #### 作業の保持

      <Constant name="cloud_ide" /> では、変更を保存するには明示的な操作が必要です。作業は3つの方法で保存されます。

      - **未保存のローカルコード -** ブラウザはコードをローカルストレージにのみ保存します。この状態では、ブランチまたはブラウザを切り替えるには、未保存の変更をコミットする必要がある場合があります。変更を保存してコミットしている場合は、未保存の変更があっても「ブランチの変更」オプションにアクセスできます。ただし、変更を保存せずにブランチを切り替えようとすると、未保存の変更が失われることを通知する警告メッセージが表示されます。

      <Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-unsaved-modal.jpg" width="85%" title="変更を保存せずにブランチを切り替えようとすると、変更が失われることを通知する警告メッセージが表示されます。"/>

      - **保存済みだがコミットされていないコード &mdash;** ファイルを保存すると、データは耐久性のある長期ストレージに保存されますが、Git には同期されません。**ブランチを変更** オプションを使用してブランチを切り替えるには、変更を「コミットして同期」または「元に戻す」必要があります。保存済みだがコミットされていないコードでは、ブランチを変更できません。これは、コミットされていない変更が失われないようにするためです。
      - **コミット済みコード &mdash;** Git プロバイダーのブランチに保存され、他の（リモート）ブランチをチェックアウトできます。

  </Expandable>

## プロジェクトのビルドとドキュメント化

- **プロジェクトのビルド、コンパイル、実行** - コマンドバーまたは**ビルド**ボタンを使用して、dbt プロジェクトの*ビルド*、*コンパイル*、*実行*、*テスト*を行うことができます。**ビルド**ボタンを使用すると、作業中のモデルをすばやくビルド、実行、またはテストできます。<Constant name="cloud_ide" /> は、モデル、テスト、シード、およびオペレーションを実行するとリアルタイムで更新されます。
  - モデルまたはテストが失敗した場合、<Constant name="cloud" /> を使用すると、dbt 呼び出しの実行ログを簡単に表示およびダウンロードして問題を修正できます。
  - dbt の [豊富なモデル選択構文](/reference/node-selection/syntax) を使用して、<Constant name="cloud" /> 内で [dbt コマンド](/reference/dbt-commands) を直接実行できます。
  - [環境変数](/docs/build/environment-variables#special-environment-variables)を活用して、<Constant name="git" />ブランチ名を動的に使用します。たとえば、ブランチ名を開発スキーマのプレフィックスとして使用します。
  - [MetricFlowコマンド](/docs/build/metricflow-commands)を実行し、[<Constant name="semantic_layer" />](/docs/use-dbt-semantic-layer/dbt-sl)を使用してプロジェクト内のメトリックを作成および管理します。

- **<Constant name="copilot" /> を使用して YAML 構成を生成する** &mdash; [dbt Copilot](/docs/cloud/dbt-copilot) は、<Constant name="cloud" /> での開発の自動化を支援する強力な人工知能 (AI) 機能です。自然言語を使用して [コードを生成](/docs/cloud/dbt-cloud-ide/develop-copilot#generate-and-edit-code) し、[リソースを生成](/docs/cloud/dbt-cloud-ide/develop-copilot#generate-resources) (ドキュメント、テスト、メトリック、セマンティックモデルなど) を <Constant name="cloud_ide" /> 内で直接生成できるため、より短時間でより多くの成果を達成できます。<Lifecycle status="self_service,managed,managed_plus" />

- **プロジェクトのドキュメントをビルドして表示する** &mdash; <Constant name="cloud_ide" /> を使用すると、コードの開発中に dbt プロジェクトのドキュメントを[ビルドして表示](/docs/explore/build-and-view-your-docs) することが可能になります。このワークフローにより、変更を本番環境にリリースする前に、プロジェクトで生成されたドキュメントがどのように表示されるかを確認できます。


## 関連ドキュメント

- [dbt プロジェクトのスタイル設定方法](/best-practices/how-we-style/0-how-we-style-our-dbt-projects)
- [ユーザーインターフェース](/docs/cloud/dbt-cloud-ide/ide-user-interface)
- [バージョン管理の基本](/docs/cloud/git/version-control-basics)
- [dbt コマンド](/reference/dbt-commands)

## FAQs

<DetailsToggle alt_header="Cloud IDE の使用には費用がかかりますか?">
いいえ、そうではありません！<a href="https://www.getdbt.com/pricing/">無料の開発者プラン</a>にご登録いただくと、<Constant name="cloud" />をご利用いただけます。このプランには開発者シートが1つ付属しています。より多くの機能をご利用になりたい場合や、開発者シートを増やしたい場合は、アカウントをStarter、Enterprise、またはEnterprise+プランにアップグレードしてください。<br />

詳しくは、<a href="https://www.getdbt.com/pricing/">dbtの料金プラン</a>をご覧ください。
</DetailsToggle>

<DetailsToggle alt_header="dbtに貢献できますか？">
<Constant name="cloud" /> はプロプライエタリ製品であるため、ソースコードはコミュニティへの貢献に利用できません。dbt エコシステムで何かを開発したい場合は、[この記事](/community/contributing/contributing-coding) で dbt パッケージ、プラグイン、dbt-core、またはこのドキュメントサイトへの貢献についてご確認ください。オープンソースへの参加は、開発者としてのレベルアップとコミュニティへの貢献に大きく貢献する素晴らしい方法です。
</DetailsToggle>

<DetailsToggle alt_header="Studio IDE、dbt CLI、dbt Core での開発の違いは何ですか?">
dbt は、<Constant name="cloud" /> の Web ベース IDE、<Constant name="cloud_cli" /> を使用したコマンドラインインターフェース、またはオープンソースの <Constant name="core" /> を使用して開発できます。いずれの環境でも dbt コマンドを実行できます。<Constant name="cloud_cli" /> と <Constant name="core" /> の主な違いは、<Constant name="cloud_cli" /> が <Constant name="cloud" /> のインフラストラクチャに合わせてカスタマイズされており、そのすべての機能と統合されていることです。

- <Constant name="cloud_ide" />: <a href="https://docs.getdbt.com/docs/cloud/about-cloud/dbt-cloud-features"><Constant name="cloud" /></a> は、IDE を使用して dbt プロジェクトを開発できる Web ベースのアプリケーションです。専用のスケジューラが組み込まれており、dbt ドキュメントをチームと簡単に共有できます。 IDE は、dbt モデルをより高速かつ確実にデプロイする方法であり、dbt プロジェクトのリアルタイム編集・実行環境を提供します。

- <Constant name="cloud_cli" />: <a href="https://docs.getdbt.com/docs/cloud/cloud-cli-installation"><Constant name="cloud_cli" /></a> を使用すると、ローカルのコマンドラインまたはコードエディタから、dbt <Constant name="cloud" /> 開発環境に対して dbt コマンドを実行できます。プロジェクト間の参照、高速で低コストのビルド、ビルド成果物の自動延期などをサポートします。

- <Constant name="core" />: <Constant name="core" /> は、無料で利用できる <a href="https://github.com/dbt-labs/dbt">オープンソース</a> ソフトウェアです。コードエディタで dbt プロジェクトをビルドし、コマンドラインから dbt コマンドを実行できます。

</DetailsToggle>
