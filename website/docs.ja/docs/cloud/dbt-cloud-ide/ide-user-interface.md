---
title: "IDE user interface"
id: ide-user-interface
description: "Develop, test, run, and build in the Cloud IDE. With the Cloud IDE, you can compile dbt code into SQL and run it against your database directly"
sidebar_label: User interface
tags: [IDE]
---

[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) は、開発者がブラウザから簡単に dbt プロジェクトを構築、テスト、実行、バージョン管理し、データガバナンスを強化できるツールです。Cloud <Constant name="cloud_ide" /> を使用すると、dbt コードを SQL にコンパイルし、データベースに対して直接実行できます。コマンドラインは必要ありません。

このページでは、ユーザーインターフェース要素の包括的な定義と用語を提供し、<Constant name="cloud_ide" /> 環境を簡単に操作できるようにします。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-basic-layout.jpg" width="90%" title="The Cloud IDE layout includes version control on the upper left, files/folders on the left, editor on the right an command/console at the bottom"/>

## 基本レイアウト

<Constant name="cloud_ide" /> はワークフローを効率化し、左側にファイルとフォルダ、右側にエディター、下部にコマンドとコンソール情報を表示する、一般的なユーザーインターフェースレイアウトを採用しています。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-side-menu.jpg" width="30%" title="The Git repo link, documentation site button, Version Control menu, and File Explorer"/>

1. **<Constant name="git" /> リポジトリ リンク &mdash;** <Constant name="cloud_ide" /> の左上にある <Constant name="git" /> リポジトリ リンクをクリックすると、同じアクティブ ブランチ上のリポジトリにアクセスできます。リポジトリ名とアクティブ ブランチ名も表示されます。
  * **注:** このリンク機能は、マルチテナント <Constant name="cloud" /> アカウントの GitHub または GitLab リポジトリでのみ利用できます。

2. **ドキュメント サイト ボタン &mdash;** Git リポジトリ リンクの横にあるドキュメント サイトのブック アイコンをクリックすると、dbt ドキュメント サイトに移動します。このサイトは、コマンド バーの `dbt docs generate` コマンドを使用して IDE で生成された最新の dbt アーティファクトに基づいています。

3. [**バージョン管理**](#editing-features) &mdash; <Constant name="cloud_ide" /> の強力なバージョン管理セクションには、<Constant name="git" /> アクションボタンや **変更** セクションなど、Git 関連の要素がすべて含まれています。

4. **ファイル <Constant name="explorer" /> &mdash;** ファイル <Constant name="explorer" /> には、リポジトリのファイルツリーが表示されます。以下の操作が可能です。
  - ファイルツリー内の任意のファイルをクリックすると、ファイルエディタでそのファイルを開きます。
  - ディレクトリ間でファイルをクリックしてドラッグすると、ファイルを移動できます。
  - ファイルを右クリックすると、ファイルの複製、ファイル名のコピー、`ref` としてコピー、名前の変更、削除などのサブメニューオプションにアクセスできます。
  - ファイル名またはフォルダ名の右側にあるファイルインジケーターを使って、変更や操作が行われた日時を確認できます。
    * 未保存 (•) — <Constant name="cloud_ide" /> は、ファイル/フォルダへの未保存の変更を検出します。
    * 変更 (M) — <Constant name="cloud_ide" /> は、既存のファイル/フォルダの変更を検出します。
    * 追加 (A) — <Constant name="cloud_ide" /> は、追加されたファイルを検出します。
    * 削除 (D) — <Constant name="cloud_ide" /> は、削除されたファイルを検出します。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-command-bar.jpg" width="100%" title="Use the Command bar to write dbt commands, toggle 'Defer', and view the current IDE status"/>

5. **コマンドバー &mdash;** <Constant name="cloud_ide" /> の左下にあるコマンドバーは、[dbt コマンド](/reference/dbt-commands) を呼び出すために使用されます。コマンドが呼び出されると、関連するログが呼び出し履歴ドロワーに表示されます。

6. **本番環境への延期 &mdash;** **本番環境への延期** トグルを使用すると、開発者は編集したモデルのみをビルド、実行、テストすることができ、それ以前のモデル（上流の親）をすべて実行してビルドする必要はありません。詳細については、[<Constant name="cloud" /> での defer の使用](/docs/cloud/about-cloud-develop-defer#defer-in-the-dbt-cloud-ide) を参照してください。

7. **ステータス ボタン &mdash;** <Constant name="cloud_ide" /> の右下にある <Constant name="cloud_ide" /> ステータス ボタンには、現在の <Constant name="cloud_ide" /> ステータスが表示されます。ステータスまたは dbt コードにエラーがあり、プロジェクトの解析が停止している場合は、ボタンが赤色に変わり、「エラー」と表示されます。エラーがない場合は、ボタンに緑色の「準備完了」ステータスが表示されます。[<Constant name="cloud_ide" /> ステータス モーダル](#modals-and-menus) にアクセスするには、このボタンをクリックします。

## 編集機能

<Constant name="cloud_ide" /> には、dbt コードの記述やチームメイトとの共同作業を容易にする便利なツールとレイアウトが備わっています。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-editing.jpg" width="90%" title="Use the file editor, version control section, and save button during your development workflow"/>

1. **ファイルエディタ &mdash;** ファイルエディタはコードを編集する場所です。開いているファイルごとにタブで領域が区切られ、保存されていないファイルはタブビューで青いドットアイコンでマークされます。保護されたプライマリ Git ブランチでは、ファイルの編集、フォーマット、lint を実行したり、dbt コマンドを実行したりできます。<Constant name="cloud_ide" /> は保護されたブランチへのコミットをブロックするため、新しいブランチに変更をコミットするように促すプロンプトが表示されます。
    * 直感的な [キーボードショートカット](/docs/cloud/dbt-cloud-ide/keyboard-shortcuts) を使用すると、あなたとチームの開発がスムーズになります。

2. **保存ボタン &mdash;** エディタには、編集可能なファイルを保存する **保存** ボタンがあります。このボタンを押すか、Command + S または Control + S のショートカットを使用すると、ファイルの内容が保存されます。コンソール セクションでコード結果をプレビューするために保存する必要はありませんが、dbt 呼び出しに変更が反映される前には保存が必要です。ファイルエディタタブには、未保存の変更には青いアイコンが表示されます。

3. **バージョン管理 -** このメニューには、<Constant name="git" /> アクションボタンを含む、すべての Git 関連要素が含まれています。このボタンは、エディタの状態に基づいて関連するアクションを更新します。たとえば、リモートの変更をプルするプロンプトの表示、元に戻されたコミット変更がある場合のコミットと同期、適切な場合のマージ/プルリクエストの作成、リモートリポジトリから削除されたブランチのプルーニングなどです。
    - <Constant name="git" /> アクションボタンのドロップダウンメニューでは、変更を元に戻したり、<Constant name="git" /> の状態を更新したり、マージ/プルリクエストを作成したり、ブランチをプルーニングしたり、ブランチを変更したりできます。
    - また、[マージの競合を解決](/docs/cloud/git/merge-conflicts)することもできます。git の詳細については、[バージョン管理の基本](/docs/cloud/git/version-control-basics#the-git-button-in-the-cloud-ide) を参照してください。
    - **バージョン管理オプション メニュー &mdash;** <Constant name="git" /> アクション ボタンの下にある **変更** セクションには、前回のコミット以降のすべてのファイルの変更が一覧表示されます。変更をクリックすると、<Constant name="git" /> 差分ビューが開き、インラインの変更を確認できます。任意のファイルを右クリックし、バージョン管理オプション メニューでファイル固有のオプションを使用することもできます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/version-control-options-menu.png" width="30%" title="Right-click edited files to access Version Control Options menu"/>


    - **「ブランチのプルーニング」** オプションを使用すると、リモートリポジトリから既に削除されているローカルブランチを削除できます。このオプションを選択すると、[ポップアップモーダル](#prune-branches-modal) が起動し、特定のローカルブランチの削除を確認できるため、ブランチ管理を整理できます。ただし、これにより現在作業中のブランチが削除されるわけではないことに注意してください。[管理対象リポジトリ](/docs/cloud/git/managed-repository) では、標準的なリモート設定がないため、ブランチのプルーニングは利用できません。この設定により、リモートブランチの削除が防止されます。

## その他の編集機能

- **ミニマップ &mdash;** ミニマップ（コードアウトライン）は、ソースコードの概要を表示し、素早いナビゲーションとコードの理解に役立ちます。ファイルのミニマップはエディターの右上に表示されます。ファイル内の別のセクションに素早く移動するには、網掛け部分をクリックします。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-minimap.jpg" width="90%" title="Use the Minimap for quick navigation and code understanding"/>

- **dbt エディタ コマンド パレット &mdash;** dbt エディタ コマンド パレットには、テキスト編集操作とそれに関連するキーボード ショートカットが表示されます。F1 キーを押すか、テキスト編集領域で右クリックして「コマンド パレット」を選択することでアクセスできます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-editor-command-palette-with-save.jpg" width="90%" title="Click F1 to access the dbt Editor Command Palette menu for editor shortcuts"/>

- **<Constant name="git" /> 差分ビュー &mdash;** **バージョン管理メニュー**の**変更**セクションでファイルをクリックすると、変更されたファイルが<Constant name="git" /> 差分ビューで開きます。エディターの左側には以前のバージョン、右側にはインライン変更が表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-git-diff-view-with-save.jpg" width="90%" title="The Git Diff View displays the previous version on the left and the changes made on the right of the Editor"/>

- **Markdown プレビュー コンソール タブ &mdash;** Markdown プレビュー コンソール タブには、リポジトリ内の .md ファイルのマークダウン コードのプレビューが表示され、コードを編集すると自動的に更新されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-markdown-with-save.jpg" width="90%" title="The Markdown Preview console tab renders markdown code below the Editor tab."/>

- **CSV プレビュー コンソール タブ &mdash;** CSV プレビュー コンソール タブには、CSV ファイルからのデータがテーブル形式で表示されます。このデータは、シード ディレクトリ内のファイルを編集すると自動的に更新されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-csv.jpg" width="90%" title="View csv code in the CSV Preview console tab below the Editor tab."/>

## コンソールセクション

ファイルエディタの下にあるコンソールセクションには、プレビュー、コンパイル、ビルド、<Term id="dag" /> の表示などのタスクに役立つさまざまなコンソールタブとボタンがあります。コンソールタブとボタンの詳細については、以下の項目を参照してください。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-console-overview.jpg" width="90%" title="The Console section is located below the File editor and has various tabs and buttons to help execute tasks"/>

1. **プレビューボタン &mdash;** 「プレビュー」ボタンをクリックすると、保存の有無にかかわらず、アクティブなファイルエディタでSQLが実行され、結果がコンソールの「結果」タブに送信されます。保存済みまたは未保存のコードの選択部分をハイライト表示して「プレビュー」ボタンをクリックすると、その部分をプレビューできます。

<details>
<summary>IDE の行制限</summary>
<Constant name="cloud_ide" /> はデフォルトの行数制限を返しますが、返されるレコード数を指定することもできます。詳細については、以下の項目を参照してください。<br /><br />
<ul>
<li><b>500 行の制限:</b> IDE が返すデータが多すぎてブラウザに問題が発生するのを防ぐため、dbt は <b>プレビュー ボタン</b> の使用時に自動的に 500 行の制限を設定します。この制限は、SQL ステートメントの末尾に <code>limit your_number</code> を追加することで変更できます。たとえば、<code>SELECT * FROM</code> table <code>limit 100</code> は最大 100 行を返します。 <code>limit your_number</code> は明示的に記述する必要があり、マクロから取得することはできないことに注意してください。</li>
<li><b>行数制限のデフォルトを変更:</b> dbt バージョン 1.6 以降では、クエリ実行時に <b>結果</b> タブに表示されるデフォルトの制限である 500 行を変更できます。設定を調整するには、表示されている行の横にある <b>行表示を変更</b> をクリックします。10,000 行を超える値には設定できないことに注意してください。ページを更新するか開発セッションを閉じると、デフォルトの制限は 500 行に戻ります。</li>
<li><b>返されるレコード数を指定:</b> IDE は、返されるレコード数を指定する <code>SELECT TOP #</code> もサポートしています。</li>
</ul>
</details>

1. **コンパイルボタン &mdash;** **コンパイル** ボタンは、保存済みまたは未保存の SQL コードをコンパイルし、[**コンパイル済みコード**] タブに表示します。

dbt v1.6 以降では、モデルへの変更を保存するときに、モデル固有のコンテキストを使用してコードをコンパイルできます。このコンテキストは、モデルのビルド時に使用するコンテキストに似ており、`{{ this }} ` や `{{ is_incremental() }}` などの便利なコンテキスト変数が含まれます。

3. **ビルドボタン &mdash;** ビルドボタンを使用すると、ファイルエディタでアクティブなモデルに関連する dbt コマンドにすばやくアクセスできます。使用できるコマンドには、dbt build、dbt test、dbt run があり、現在のリソースのみ、リソースとその上流の依存関係、リソースとその下流の依存関係、またはすべての依存関係を含むリソースを含めるオプションがあります。このメニューは、すべての実行可能ノードで使用できます。

4. **Lint ボタン** &mdash; **Lint** ボタンをクリックすると、ファイルエディタ内のアクティブなファイルに対して [linter](/docs/cloud/dbt-cloud-ide/lint-format) が実行されます。linter はコード内の構文エラーやスタイルの問題をチェックし、結果を [**コード品質**] タブに表示します。

5. **dbt Copilot** &mdash; [dbt Copilot](/docs/cloud/dbt-copilot) は、ドキュメント、テスト、セマンティックモデルを自動的に生成できる強力な人工知能エンジンです。<Lifecycle status="self_service,managed,managed_plus" />

6. **結果タブ &mdash;** 結果コンソールタブには、最新のプレビュー結果が表​​形式で表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/results-console-tab.jpg" width="90%" title="Preview results show up in the Results console tab"/>

7. **コード品質タブ &mdash;** コード品質タブには、ファイルエディタ内のアクティブファイルに対するリンターの結果が表示されます。コードエラーの確認、コード品質の可視化と管理、使用されているSQLFluffのバージョン表示が可能です。

8. **コンパイル済みコードタブ &mdash;** 「コンパイル」ボタンを実行すると、コンパイル済みのコードが生成されます。「コンパイル済みコード」タブには、ファイルエディタ内のアクティブファイルのコンパイル済みSQLコードが表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/compiled-code-console-tab.jpg" width="90%" title="Compile results show up in the Compiled Code tab"/>

9. **系統タブ &mdash;** ファイルエディタの系統タブには、アクティブモデルの系統（<Term id="dag" />）が表示されます。デフォルトでは、両方向に2段階の系統（`2+model_name+2`）が表示されますが、+model+（完全なDAG）に変更できます。系統を使用するには、次の手順に従います。
    - DAG内のノードをダブルクリックして、そのファイルを新しいタブで開きます。
    - ノード選択構文を使用してDAGを拡大または縮小します。
    - 注：`--exclude`フラグはサポートされていません。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/lineage-console-tab.jpg" width="90%" title="View resource lineage in the Lineage tab"/>

## 呼び出し履歴

呼び出し履歴ドロワーは、IDE での dbt 呼び出しに関する情報を保存します。`dbt run` などの dbt コマンドを実行すると、関連するログが呼び出し履歴ドロワーに表示されます。

ドロワーは複数の方法で開くことができます。
- ページ左下のコマンドバーの横にある `^` アイコンをクリックする
- dbt コマンドを入力して Enter キーを押す
- または、Ctrl キーとバックティックキー (または Ctrl + `) を押す

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-inv-history-drawer.jpg" width="90%" title="The Invocation History Drawer returns a log and detail of all your dbt invocations."/>

1. **呼び出し履歴リスト &mdash;** 呼び出し履歴ドロワーの左側のパネルには、<Constant name="cloud_ide" /> 内の以前の呼び出しのリスト（コマンド、ブランチ名、コマンドのステータス、経過時間など）が表示されます。

2. **呼び出しサマリー &mdash;** **システムログ** の上にある呼び出しサマリーには、呼び出し履歴リストから選択したコマンドに関する情報（コマンド、そのステータス（実行中の場合は「実行中」）、コマンド実行時にアクティブだった Git ブランチ、コマンドの呼び出し時刻など）が表示されます。

3. **システムログ切り替え &mdash;** 呼び出しサマリーの下にあるシステムログ切り替えを使用すると、呼び出されたコマンド全体の完全な標準出力ログとデバッグログを表示できます。

4. **コマンド コントロール ボタン -** 右側にあるコマンド コントロール ボタンを使用して、呼び出しを制御し、選択した実行をキャンセルまたは再実行します。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-results.jpg" width="90%" title="The Invocation History list displays a list of previous invocations in the IDE"/>

5. **ノード概要タブ &mdash;** 結果ステータスタブをクリックすると、対応するステータスに基づいてノードステータスリストがフィルタリングされます。使用可能なステータスは、Pass（ノードの呼び出しが成功）、Warn（警告付きでテスト実行）、Error（データベースエラーまたはテスト失敗）、Skip（上流のエラーによりノードが実行されなかった）、Queued（まだ実行されていないノード）です。

6. **ノード結果トグル &mdash;** dbt コマンドを実行すると、実行された各ノードに関する情報がノード結果トグルに表示されます。このトグルには、サマリーとデバッグログが含まれます。ノード結果リストには、コマンド中に呼び出されたすべてのノードがリストされます。

7. **ノード結果リスト &mdash;** ノード結果リストには、dbt 実行で使用されたすべてのノード結果が表示され、結果ステータスタブをクリックしてフィルタリングできます。

## モーダルとメニュー
メニューとモーダルを使用して <Constant name="cloud_ide" /> を操作し、開発ワークフローに役立つ便利なオプションにアクセスできます。

- #### エディタータブメニュー
  開いているエディタータブを操作するには、任意のタブを右クリックして、ファイルタブメニューの便利なオプションにアクセスします。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/editor-tab-menu-with-save.jpg" width="90%" title=" Right-click a tab to view the Editor tab menu options"/>

- #### ファイル検索
  ファイルナビゲーションメニューを使用すると、ファイルを簡単に検索したり、ファイル間を移動したりできます。ファイルナビゲーションメニューは、Command + O または Control + O を押すか、ファイルメニューの 🔍 アイコンをクリックすることでアクセスできます。<Constant name="explorer" />

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-file-search-with-save.jpg" width="100%" title="The Command History returns a log and detail of all your dbt invocations."/>

- #### グローバルコマンドパレット
  グローバルコマンドパレットには、git アクション、特殊な dbt コマンド、コンパイル、プレビューなどの <Constant name="cloud_ide" /> を操作するための便利なショートカットが用意されています。メニューを開くには、Command + P または Control + P を押します。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-global-command-palette-with-save.jpg" width="100%" title="The Command History returns a log and detail of all your dbt invocations."/>

- #### <Constant name="cloud_ide" /> ステータスモーダル
  <Constant name="cloud_ide" /> ステータスモーダルには、サーバーの現在のエラーメッセージとデバッグログが表示されます。また、<Constant name="cloud_ide" /> を再起動するオプションも含まれています。<Constant name="cloud_ide" /> ステータスボタンをクリックしてこのモーダルを開いてください。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-status-modal-with-save.jpg" width="90%" title="The Command History returns a log and detail of all your dbt invocations."/>

- #### 新しいブランチにコミットする
  保護されたプライマリ Git ブランチを直接編集し、準備ができたらその変更を新しいブランチにコミットします。

  <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/create-new-branch.png" width="70%" title="Commit changes to a new branch"/>

- #### 変更をコミットするモーダル
  変更をコミットするモーダルは、<Constant name="git" /> アクションボタンからアクセスでき、すべての変更をコミットできます。また、バージョン管理オプションメニューから個々の変更をコミットすることもできます。コミットメッセージを入力したら、モーダルを使用して選択した変更をコミットし、同期できます。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/commit-changes-modal.png" width="90%" title="The Commit Changes modal is how users commit changes to their branch."/>

- #### ブランチ変更モーダル
  ブランチ変更モーダルを使用すると、<Constant name="cloud_ide" /> 内の Git ブランチを切り替えることができます。**ブランチ変更** リンク、または **バージョン管理** メニューの **<Constant name="git" /> アクション** ボタンからアクセスできます。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/change-branch-modal.png" width="90%" title="The Commit Changes modal is how users change their branch."/>

- #### ブランチのプルーニング モーダル
  「ブランチのプルーニング」モーダルを使用すると、リモートリポジトリから削除されたローカルブランチを削除して、ブランチ管理を整理できます。これは、[**バージョン管理** メニュー](#editing-features) の **<Constant name="git" /> アクション** ボタンからアクセスできます。ただし、これにより現在作業中のブランチが削除されるわけではありません。管理対象リポジトリでは、一般的なリモート設定がないため、ブランチのプルーニングは利用できません。この設定により、リモートブランチの削除が防止されます。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/prune-branch-modal.jpg" width="60%" title="The Prune branches modal allows users to delete local branches that have already been deleted from the remote repository."/>

- #### コミットされていない変更を元に戻すモーダル
  コミットされていない変更を元に戻すモーダルは、IDE での変更を元に戻すためのものです。バージョン管理オプションメニューの上にある「ファイルを元に戻す」オプション、または IDE に保存済みでコミットされていない変更がある場合は「Git アクション」ボタンからアクセスできます。

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/revert-uncommitted-changes-with-save.jpg" width="90%" title="The Commit Changes modal is how users change their branch."/>

- #### <Constant name="cloud_ide" /> オプションメニュー
  <Constant name="cloud_ide" /> オプションメニューは、<Constant name="cloud_ide" /> の右下にある3点メニューをクリックすると表示されます。このメニューには、以下のグローバルオプションが含まれています。

  * 見やすさを向上させるために、ダークモードとライトモードを切り替える
  * <Constant name="cloud_ide" /> を再起動する
  * リポジトリをリモートにロールバックして、Gitの状態を更新し、ステータスの詳細を表示する
  * <Constant name="cloud_ide" /> ステータスモーダルを含む、ステータスの詳細を表示する

  <Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-options-menu-with-save.jpg" width="90%" title="Access the IDE Options menu to switch to dark or light mode, restart the IDE, rollback to remote, or view the IDE status"/>
