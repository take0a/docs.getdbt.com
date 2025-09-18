
VS Code および Cursor 用の dbt 拡張機能は、dbt 開発ワークフローを効率化します。dbt 拡張機能は <Constant name="fusion_engine" /> を搭載しています。

## 前提条件

拡張機能を使用するには、以下の前提条件を満たす必要があります。

- dbt拡張機能を使用するには、<Constant name="fusion_engine" />のインストールが必要です。<Constant name="fusion" />のインストールは拡張機能のインストールプロセスの一部ですが、拡張機能のインストール前またはインストール後に、このワークフローとは別に[手動でインストール](/docs/fusion/install-fusion)することもできます。
- [VS Code](https://code.visualstudio.com/)または[Cursor](https://www.cursor.com/en)コードエディターを使用していること。
- サードパーティ製のdbt拡張機能を使用していない（または無効にしている）こと。
- macOS<!--, Windows,-->またはLinuxベースのコンピューターを使用していること。

## インストール手順

:::note

これは唯一の公式dbt Labs VS Code拡張機能です。問題を回避するため、インストール前にサードパーティ製のdbt拡張機能を無効化またはアンインストールしてください。

最新情報については、[Fusion Diaries](https://github.com/dbt-labs/dbt-fusion/discussions/categories/announcements)をご覧ください。

::: 

VS Code では:

1. エディターの **Extensions** タブに移動し、「dbt」を検索します。発行元が `dbtLabsInc` または `dbt Labs Inc` である拡張機能を見つけます。**Install** をクリックします。
<Lightbox src="/img/docs/extension/extension-marketplace.png" width="60%" title="拡張機能を検索"/>
2. VS Code 環境で dbt プロジェクトをまだ開いていない場合は開きます。現在のワークスペースに追加されていることを確認してください。エディターのステータスバーに **dbt Extension** ラベルが表示されていれば、拡張機能は正常にインストールされています。この **dbt Extension** ラベルにマウスポインターを合わせると、拡張機能の診断情報が表示されます。
<Lightbox src="/img/docs/extension/dbt-extension-statusbar.png" width="60%" title="「dbt Extension」ラベルが表示されている場合、拡張機能は有効化されています。"/>
3. dbt拡張機能が有効化されると、お使いのオペレーティングシステムに適したdbt Language Serverのダウンロードが自動的に開始されます。
<Lightbox src="/img/docs/extension/extension-lsp-download.png" width="60%" title="dbt Language Serverが自動的にインストールされます。"/>
4. dbt Fusionエンジンがまだマシンにインストールされていない場合は、拡張機能によってダウンロードとインストールを求めるメッセージが表示されます。通知に表示される手順に従ってインストールを完了してください。
<Lightbox src="/img/docs/extension/install-dbt-fusion-engine.png" width="60%" title="プロンプトに従ってdbt Fusionエンジンをインストールします"/>
5. VS Code拡張機能の[アップグレードツール](#upgrade-to-fusion)を実行して、dbtプロジェクトがFusionに対応していることを確認し、エラーや非推奨の修正に役立てます。
6. これで準備完了です！dbt拡張機能の使用方法の詳細については、[dbt拡張機能について](/docs/about-dbt-extension)をご覧ください。
<Lightbox src="/img/docs/extension/kitchen-sink.png" width="60%" title="拡張機能内の系統とコンパイル済みコードの表示"/>

## Fusionにアップグレード

:::note

<Constant name="fusion_engine" /> をすでに実行している場合、アップグレード ツールを使用するにはバージョン `2.0.0-beta.66` 以上を使用する必要があります。

:::

dbt 拡張機能には、<Constant name="fusion" /> の設定と dbt プロジェクトの更新プロセスをガイドする組み込みのアップグレードツールが用意されており、すべての機能をサポートし、非推奨のコードがあれば修正できます。プロセスを開始するには、次の手順に従います。

1. VS Code の左側のメニューから、**dbt ロゴ** をクリックします。
2. 表示されたペインで、**開始** セクションを開き、**開始** ボタンをクリックします。

<Lightbox src="/img/docs/extension/fusion-onboarding-experience.png" title="dbt 拡張機能のヘルプペインとアップグレードアシスタント。" width="60%" />

CLI ウィンドウを開いて次のコマンドを実行し、このプロセスを手動で開始することもできます。

```
dbt init --fusion-upgrade
```

これによりアップグレードツールが起動し、一連のプロンプトが表示され、Fusion のアップグレード手順が案内されます。
- **既存の dbt プラットフォーム アカウントをお持ちですか？**: `Y` と回答すると、拡張機能を登録するための dbt プラットフォーム プロファイルのダウンロード手順が表示されます。`N` と回答すると、次のステップに進みます。
- **dbtf init を実行する準備はできましたか？** (`profiles.yml` ファイルが存在しない場合): データ ウェアハウスへの接続を含む、dbt の構成プロセスを実行します。
- **dbtf debug を実行する準備はできましたか？** (`profiles.yml` ファイルが存在する場合): プロジェクトが正しく構成され、データ ウェアハウスに接続できることを検証します。
- **dbtf parse を実行する準備はできましたか？**: dbt プロジェクトが解析され、<Constant name="fusion" /> との互換性が確認されます。
    - 解析中に問題が発生した場合は、[dbt-autofix](https://github.com/dbt-labs/dbt-autofix?tab=readme-ov-file#installation) ツールを実行してエラーを解決するオプションが表示されます。アップグレードプロセス中にツールを実行しない場合は、後でいつでも実行したり、手動でエラーを修正したりできます。ただし、エラーが解決されるまでアップグレードツールは続行できません。
- **「dbtf compile -static-analysis off」を実行しますか？** (解析が成功した場合にのみ実行): dbt Core を模倣し、静的解析なしでプロジェクトをコンパイルします。このコンパイルでは Jinja を SQL に変換するだけなので、<Constant name="fusion" /> の高度な SQL 理解は一時的に無効になります。
- **「dbtf compile」を実行しますか？**: 完全な <Constant name="fusion" /> 静的解析を使用してプロジェクトをコンパイルします。 SQL コードがウェアハウスのテーブルと列のコンテキストで有効であることを確認します。

<Lightbox src="/img/docs/extension/fusion-onboarding-complete.png" title="プロジェクトの dbt Fusion エンジンへのアップグレードが完了したときに表示されるメッセージ。" width="60%" />

アップグレードが完了したら、<Constant name="fusion_engine" /> が提供するすべての機能をすぐにご利用いただけます。

## 拡張機能の登録

dbt拡張機能をインストールしてから14日以内に登録を完了する必要があります。登録方法は2通りあります。

- dbtアカウントをお持ちでない方は、オンライン登録フォームから簡単に登録できます。初回インストール時は、お名前とメールアドレスを入力するだけで登録が完了します。2回目以降のインストールでは、拡張機能を使用するために[dbtアカウント登録プロセス](#dbtアカウントへのアクセス)をすべて完了する必要があります。
- dbtアカウントをお持ちの方は、`dbt_cloud.yml`認証情報ファイルを使用してアカウントを接続できます。

VS Code拡張機能は、組織で最大15ユーザーまで無料でご利用いただけます。

### 新規ユーザー登録

dbtアカウントをお持ちでない場合は、登録が必要です。登録は1分ほどで完了します！
1. エディターの登録プロンプトをクリックします。
<Lightbox src="/img/docs/extension/registration-prompt.png" width="60%" title="VS Codeの拡張機能登録プロンプト。"/>
2. プロンプトが表示されたら承認し、ブラウザでリンクを開きます。
3. 登録フォームに必要事項を入力し、[**続行**] をクリックします。
<Lightbox src="/img/docs/extension/registration-screen.png" width="60%" title="ブラウザの拡張機能登録ページ。"/>
4. 確認リンクが記載されたメールが届きます。リンクをクリックすると登録が完了します。

### dbtアカウントへのアクセス

dbt拡張機能の利用登録を済ませると、dbtアカウントを簡単に作成できます。以下の手順に従ってアカウント設定を完了してください（_注: dbt拡張機能の利用には必須ではありません_）。

1. [us1.dbt.com](https://us1.dbt.com) にアクセスし、「**パスワードをお忘れですか？**」をクリックします。
2. dbt拡張機能の登録に使用したメールアドレスを入力し、「**続行**」をクリックします。
3. メールで確認リンクを確認し、パスワードリセットの手順に従ってアカウントのパスワードを設定します。

dbt開発者アカウントが有効化されましたので、<Constant name="dbt_platform" /> の機能にアクセスできるようになりました。新しいマシンで dbt 拡張機能を設定する必要がある場合は、[既存の dbt アカウントで登録する](#register-with-an-existing-dbt-account) に記載されている手順を使用して登録キーを再度ダウンロードすることもできます。

### 既存のdbtアカウントで登録する

<!-- This anchor is linked from the VS Code registration page. Please do not change it -->

dbt アカウントを既にお持ちの場合は、dbt 拡張機能を使用するために再登録する必要はありません。dbt 拡張機能は、`dbt_cloud.yml` ファイルを使用して dbt プラットフォームで認証できます。このファイルが `~/.dbt/` フォルダに存在する場合、登録フローは登録時に自動的にこのファイルの使用を試みます。`~/.dbt/dbt_cloud.yml` ファイルをダウンロードしていない場合は、以下の手順を参照してください。

<Expandable alt_header="Fusion が有効になっている dbt アカウントの場合">

1. dbt アカウントにログインします。
2. 左側のメニューの下部にあるアカウント名をクリックし、**アカウント設定** をクリックします。
3. **あなたのプロフィール** セクションで、**VS Code 拡張機能** をクリックします。
4. **資格情報の設定** セクションで、**資格情報のダウンロード** をクリックします。これにより、`dbt_cloud.yml` ファイルがダウンロードされます。
<Lightbox src="/img/docs/extension/download-registration-2.png" width="60%" title="dbt_cloud.yml ファイルをダウンロードして登録を完了してください。"/>
5. ダウンロードした `dbt_cloud.yml` ファイルを `~/.dbt/` ディレクトリに移動します。
6. VS Code で登録を更新するには、コマンドパレット（`ctrl+shift+P` (<!--Windows/-->Linux) または `cmd+shift+p` (macOS)）を開き、「dbt: Register dbt extension」を選択して登録を完了します。

</Expandable>

<Expandable alt_header="Fusion が有効になっていない dbt アカウントの場合">

1. dbt アカウントにログインします。
2. 左側のメニューの下部にあるアカウント名をクリックし、[**アカウント設定**] をクリックします。
3. [**プロフィール**] セクションで、[**CLI**] をクリックします。
4. [**クラウド認証の構成**] セクションで、[**CLI 構成ファイルをダウンロード**] をクリックします。これにより、`dbt_cloud.yml` ファイルがダウンロードされます。
<Lightbox src="/img/docs/extension/download-registration.png" width="60%" title="dbt_cloud.yml ファイルをダウンロードして登録を完了します。"/>
5. ダウンロードした `dbt_cloud.yml` ファイルを `~/.dbt/` ディレクトリに移動します。
6. VS Code で登録を更新するには、コマンド パレット (`ctrl+shift+P` (<!--Windows/-->Linux) または `cmd+shift+p` (macOS)) を開き、「dbt: Register dbt extension」を選択して登録を完了します。

</Expandable>

## トラブルシューティング
<!-- This anchor is linked from the  VS Code extension. Please do not change it -->

#### dbt プラットフォームの設定

クラウドベースの dbt プラットフォームユーザーで、`dbt_project.yml` ファイルに `dbt-cloud:` 設定があり、[dbt Mesh](/docs/mesh/about-mesh) も使用している場合は、プロジェクト ID を設定する必要があります。

```yaml
dbt-cloud:
  project-id: 12345 # Required
```

これを正しく構成しないと、クロスプラットフォーム参照が正しく解決されず、dbt コマンドの実行時にエラーが発生します。

#### 一般的なトラブルシューティングのヒント

dbt拡張機能が正常に有効化されている場合、エディターの左下にあるステータスバーに「dbt Extension」というラベルが表示されます。**dbt Extension** ボタンをクリックすると、dbt拡張機能の診断情報を表示できます。

dbt拡張機能のラベルが表示されない場合は、dbt拡張機能が正常にインストールされていない可能性があります。その場合は、拡張機能をアンインストールし、エディターを再起動してから再インストールしてみてください。

注: VS Code では、ステータスバーの項目を「非表示」にすることができます。エディターのステータスバーを右クリックして、**dbt Extension** ステータスバーのラベルが非表示になっているかどうかを確認してください。右クリックメニューに **dbt Extension** が表示されていれば、拡張機能は正常にインストールされています。

#### dbt LSP 機能が表示されない

エディタで dbt LSP 機能が表示されない場合は、まず上記の一般的なトラブルシューティング手順をご確認ください。dbt 拡張機能が正しくインストールされていることを確認しても、dbt Language Server 機能（オートコンプリート、定義への移動、ホバーテキストなど）が表示されない場合は、以下の手順に従ってください。
- エディタの拡張機能ページで dbt 拡張機能のバージョンを確認してください。dbt 拡張機能の最新バージョンを使用していることを確認してください。
- `cmd+shift+P` (macOS) または `ctrl+shift+P` (<!--Windows/-->Linux) を押して `dbt: Reinstall dbt LSP` コマンドを選択し、dbt Language Server を再インストールしてみてください。

#### サポートされていない dbt バージョン

dbt のバージョンがサポートされていないことを示すエラーメッセージが表示された場合は、環境に問題がある可能性があります。
- VS Code 設定で **dbt パス** の設定を確認してください。このパスが設定されている場合は、有効な <Constant name="fusion_engine" /> 実行可能ファイルが指定されていることを確認してください。
- 必要に応じて、[Fusion CLI のインストール](/docs/fusion/install-fusion) の手順に従って、<Constant name="fusion_engine" /> を直接インストールすることもできます。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />
