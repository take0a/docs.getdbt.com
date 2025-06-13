---
title: Install the dbt VS Code extension
id: install-dbt-extension
description: "Installation instructions for the dbt extension."
sidebar_label: "Install the dbt extension"
---

# dbt VS Code拡張機能をインストールする <Lifecycle status="beta" />

VS CodeとCursor用のdbt拡張機能は、dbt開発ワークフローを効率化します。dbt拡張機能は、dbt Fusionエンジンを搭載しています。

## 前提条件

拡張機能を使用するには、以下の前提条件を満たす必要があります。

- [VS Code](https://code.visualstudio.com/) または [Cursor](https://www.cursor.com/en) コードエディターを使用していること。
- サードパーティ製の dbt 拡張機能を使用していない（または無効化している）こと。
- macOS、Windows、または Linux ベースのコンピューターを使用していること。
- dbt 拡張機能を使用するには、dbt Fusion エンジンのインストールが必要です。Fusion のインストールは、拡張機能のインストールプロセスの一部です。

## インストール手順

:::note

これは唯一の公式dbt Labs VS Code拡張機能です。問題を回避するため、インストール前にサードパーティ製のdbt拡張機能を無効にするかアンインストールしてください。

:::

import InstallExtension from '/snippets.ja/_install-dbt-extension.md'; 

<InstallExtension/>

## 拡張機能の登録

dbt拡張機能をインストールしてから14日以内に登録を完了する必要があります。登録方法は2通りあります。

- dbtアカウントをお持ちでない方は、オンライン登録フォームから簡単に登録できます。初回インストール時は、お名前とメールアドレスを入力するだけで登録が完了します。2回目以降のインストールでは、拡張機能を使用するために[dbtアカウント登録プロセス](#dbtアカウントへのアクセス)をすべて完了する必要があります。
- dbtアカウントをお持ちの方は、`dbt_cloud.yml`認証情報ファイルを使用してアカウントを接続できます。

VS Code拡張機能は、組織で最大15ユーザーまで無料でご利用いただけます。

### 新規ユーザー登録

dbtアカウントをお持ちでない場合は、登録が必要です。登録は1分ほどで完了します！
1. エディターの登録プロンプトをクリックします。
     <Lightbox src="/img/docs/extension/registration-prompt.png" width="60%" title="The extension registration prompt in VS Code."/>
2. ブラウザでリンクを開くためのプロンプトがあればそれに従います。
3. 登録フォームに記入し、「**続行**」をクリックします。
    <Lightbox src="/img/docs/extension/registration-screen.png" width="60%" title="The extension registration page in the browser."/>
4. 確認リンクが記載されたメールが届きます。クリックすると登録が完了します。

### dbtアカウントへのアクセス

dbt拡張機能の利用登録を済ませると、dbtアカウントを簡単に作成できます。以下の手順に従ってアカウントの設定を完了してください（_注: dbt拡張機能の利用には必須ではありません_）。

1. [us1.dbt.com](https://us1.dbt.com) にアクセスし、「**パスワードをお忘れですか？**」をクリックします。
2. dbt拡張機能の登録に使用したメールアドレスを入力し、「**続行**」をクリックします。
3. メールに記載されている確認リンクを確認し、パスワードリセットの手順に従ってアカウントのパスワードを設定します。

これでdbt開発者アカウントが有効化され、dbtプラットフォームの機能にアクセスできるようになりました。新しいマシンでdbt拡張機能を設定する必要がある場合は、以下の「既存のdbtアカウントで登録する」の手順に従って登録キーを再ダウンロードすることもできます。

### 既存のdbtアカウントで登録する

<!-- This anchor is linked from the VS Code registration page. Please do not change it -->

dbtアカウントを既にお持ちの場合は、dbt拡張機能を使用するために再登録する必要はありません。dbt拡張機能は、`dbt_cloud.yml`ファイルを使用してdbtプラットフォームで認証できます。このファイルが`~/.dbt/`フォルダに存在する場合、登録フローは自動的にこのファイルの使用を試みます。`~/.dbt/dbt_cloud.yml`ファイルをダウンロードしていない場合は、以下の手順に従ってください。

<Expandable alt_header="Fusionが有効になっているdbtアカウントの場合">

1. dbt アカウントにログインします。
2. 左側のメニューの下部にあるアカウント名をクリックし、「**アカウント設定**」をクリックします。
3. 「**プロフィール**」セクションで、「**VS Code 拡張機能**」をクリックします。
4. 「**資格情報の設定**」セクションで、「**資格情報のダウンロード**」をクリックします。これにより、`dbt_cloud.yml` ファイルがダウンロードされます。
    <Lightbox src="/img/docs/extension/download-registration-2.png" width="60%" title="Download the dbt_cloud.yml file to complete registration."/>
5. ダウンロードした `dbt_cloud.yml` ファイルを `~/.dbt/` ディレクトリに移動します。
6. VS Code で登録を更新するには、コマンドパレット (`ctrl+shift+P` (Windows/Linux) または `cmd+shift+p` (macOS)) を開き、「dbt: Register dbt extension」を選択して登録を完了します。

</Expandable>

<Expandable alt_header="Fusionが有効になっていないdbtアカウントの場合">

1. dbt アカウントにログインします。
2. 左側のメニューの下部にあるアカウント名をクリックし、「**アカウント設定**」をクリックします。
3. 「**プロフィール**」セクションで、「**CLI**」をクリックします。
4. 「**クラウド認証の構成**」セクションで、「**CLI 構成ファイルをダウンロード**」をクリックします。これにより、「dbt_cloud.yml」ファイルがダウンロードされます。
    <Lightbox src="/img/docs/extension/download-registration.png" width="60%" title="Download the dbt_cloud.yml file to complete registration."/>
5. ダウンロードした `dbt_cloud.yml` ファイルを `~/.dbt/` ディレクトリに移動します。
6. VS Code で登録を更新するには、コマンドパレット (`ctrl+shift+P` (Windows/Linux) または `cmd+shift+p` (macOS)) を開き、「dbt: Register dbt extension」を選択して登録を完了します。

</Expandable>

## トラブルシューティング
<!-- This anchor is linked from the  VS Code extension. Please do not change it -->

#### 一般的なトラブルシューティングのヒント

dbt拡張機能が正常に有効化されている場合、エディターの左下にあるステータスバーに「dbt拡張機能」というラベルが表示されます。**dbt拡張機能** ボタンをクリックすると、dbt拡張機能の診断情報を表示できます。

dbt拡張機能のラベルが表示されない場合は、dbt拡張機能が正常にインストールされていない可能性があります。その場合は、拡張機能をアンインストールし、エディターを再起動してから再インストールしてみてください。

注: VS Code では、ステータスバーの項目を「非表示」にすることができます。エディターのステータスバーを右クリックして、**dbt拡張機能** のステータスバーラベルが非表示になっているかどうかを確認してください。右クリックメニューに **dbt拡張機能** が表示されている場合は、拡張機能が正常にインストールされています。

#### dbt LSP 機能が表示されない

エディタで dbt LSP 機能が表示されない場合は、まず上記の一般的なトラブルシューティング手順をご確認ください。dbt 拡張機能が正しくインストールされていることを確認しても、dbt Language Server 機能（オートコンプリート、定義への移動、ホバーテキストなど）が表示されない場合は、以下の手順に従ってください。
- エディタの拡張機能ページで dbt 拡張機能のバージョンを確認してください。dbt 拡張機能の最新バージョンを使用していることを確認してください。
- `cmd+shift+P` (macOS) または `ctrl+shift+P` (Windows/Linux) を押して `dbt: Reinstall dbt LSP` コマンドを選択し、dbt Language Server を再インストールしてみてください。

#### サポートされていない dbt バージョン

dbt のバージョンがサポートされていないことを示すエラーメッセージが表示された場合は、環境に問題がある可能性があります。
- VS Code 設定で **dbt Path** 設定を確認してください。このパスが設定されている場合は、有効な dbt Fusion エンジン実行可能ファイルを指していることを確認してください。
- 必要に応じて、[Fusion CLI のインストール](/docs/fusion/install-fusion) の手順に従って、dbt Fusion エンジンを直接インストールすることもできます。