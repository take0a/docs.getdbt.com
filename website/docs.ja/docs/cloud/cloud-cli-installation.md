---
title: Install dbt CLI 
sidebar_label: "Installation"
id: cloud-cli-installation
description: "Instructions for installing and configuring dbt CLI"
pagination_next: "docs/cloud/configure-cloud-cli"
---

<Constant name="cloud" /> はコマンドライン (CLI) を使用した開発をネイティブにサポートしており、チームメンバーはより柔軟かつ共同作業を行いながら開発に貢献できます。<Constant name="cloud" /> CLI を使用すると、ローカルコマンドラインから <Constant name="cloud" /> 開発環境に対して dbt コマンドを実行できます。

dbt コマンドは <Constant name="cloud" /> のインフラストラクチャに対して実行され、以下の利点があります。

* <Constant name="cloud" /> プラットフォームにおける安全な認証情報ストレージ
* ビルド成果物の Cloud プロジェクトの運用環境への [自動延期](/docs/cloud/about-cloud-develop-defer)
* より高速で低コストのビルド
* dbt Mesh のサポート ([プロジェクト間 `ref`](/docs/mesh/govern/project-dependencies))
* 今後数か月以内にリリース予定のプラットフォームの大幅な改善

<Lightbox src="/img/docs/dbt-cloud/cloud-cli-overview.jpg" title="Diagram of how the dbt CLI works with dbt's infrastructure to run dbt commands from your local command line." />

## 前提条件

<Constant name="cloud" /> CLI は、すべての [デプロイリージョン](/docs/cloud/about-cloud/access-regions-ip-addresses) およびマルチテナント アカウントとシングルテナント アカウントの両方で使用できます。

## dbt CLI をインストールする

以下のいずれかの方法で、コマンドラインから <Constant name="cloud" /> CLI をインストールできます。

<details>
<summary>インストールの手順を説明したビデオチュートリアルをご覧ください。</summary>

<LoomVideo id="dd80828306c5432a996d4580135041b6?sid=fe1895b7-1281-4e42-9968-5f7d11768000"/>

</details>

<Tabs queryString="install">

<TabItem value="brew" label="macOS (brew)">

始める前に、コードエディタまたはコマンドラインターミナルに[Homebrewがインストールされている](http://brew.sh/)ことを確認してください。オペレーティングシステムでパスの競合が発生した場合は、[FAQ](#faqs)を参照してください。

1. 次のコマンドを実行して、dbt Coreがまだインストールされていないことを確認します:
  
  ```bash
  which dbt
  ```
  
  出力が「dbt not found」の場合、インストールされていないことが確認できます。

:::tip Run `pip uninstall dbt` to uninstall dbt Core

他の方法で dbt Core をグローバルにインストールした場合は、続行する前にまずアンインストールしてください:

```bash
pip uninstall dbt
```

:::

2. Homebrew で <Constant name="cloud_cli" /> をインストールします。

   - まず、パッケージ用の別リポジトリである `dbt-labs` タップを Homebrew から削除します。これにより、Homebrew がこのリポジトリからパッケージをインストールできなくなります。
      ```bash
      brew untap dbt-labs/dbt
   - 次に、<Constant name="cloud_cli" /> をパッケージとして追加してインストールします。
      ```bash
      brew tap dbt-labs/dbt-cli
      brew install dbt
      ```
      複数のタップがある場合は、`brew install dbt-labs/dbt-cli/dbt` を使用します。

3. コマンドラインで「dbt --help」を実行してインストールを確認してください。以下の出力が表示されれば、インストールは正しく行われています:

      ```bash
      The dbt CLI - an ELT tool for running SQL transformations and data models in dbt...
      ```

     この出力が表示されない場合は、pyenv または venv が無効になっており、グローバル dbt バージョンがインストールされていないことを確認してください。
   
   * 環境の起動時に`dbt deps`コマンドを実行する必要がなくなりました。この手順は以前は初期化時に必要でした。ただし、`packages.yml`ファイルに変更を加えた場合は、引き続き`dbt deps`を実行する必要があります。

4. `git clone` を使用して、リポジトリをローカルコンピュータにクローンします。たとえば、HTTPS 形式を使用して GitHub リポジトリをクローンするには、`git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY` を実行します。

5. リポジトリをクローンしたら、<Constant name="cloud" /> プロジェクトの <Constant name="cloud_cli" /> を [configure](/docs/cloud/configure-cloud-cli) します。これにより、[`dbt environment show`](/reference/commands/dbt-environment) を使用して <Constant name="cloud" /> 構成を表示したり、`dbt compile` を使用してプロジェクトをコンパイルし、モデルとテストを検証したりといった dbt コマンドを実行できるようになります。また、リポジトリにファイルを追加、編集、同期することもできます。

</TabItem>

<TabItem value="windows" label="Windows (native executable)">

ご使用のオペレーティング システムでパスの競合が発生した場合は、[FAQ](#faqs) を参照してください。

1. [GitHub](https://github.com/dbt-labs/dbt-cli/releases) から、ご使用のプラットフォーム向けの最新の Windows リリースをダウンロードします。

2. `dbt.exe` 実行ファイルを dbt プロジェクトと同じフォルダに解凍します。

:::info

上級ユーザーは、以下の手順で複数のプロジェクトで同じ <Constant name="cloud" /> CLI を使用するように設定できます。

1. 実行ファイル (`.exe`) を「Program Files」フォルダに配置する
2. [Windows PATH 環境変数に追加する](https://medium.com/@kevinmarkvi/how-to-add-executables-to-your-path-in-windows-5ffa4ce61a53)
3. 必要な場所に保存する

VS Code を使用している場合は、変更した環境変数を反映させるために再起動する必要があることに注意してください。
:::

4. コマンドラインで「./dbt --help」を実行してインストールを確認します。以下の出力が表示されれば、インストールは正しく行われています。

      ```bash
      The dbt CLI - an ELT tool for running SQL transformations and data models in dbt...
      ```

      この出力が表示されない場合は、pyenv または venv が無効化されていること、およびグローバル dbt バージョンがインストールされていないことを確認してください。

   * 環境の起動時に `dbt deps` コマンドを実行する必要がなくなったことに注意してください。この手順は以前は初期化時に必要でした。ただし、`packages.yml` ファイルに変更を加えた場合は、引き続き `dbt deps` を実行する必要があります。

5. `git clone` を使用して、リポジトリをローカルコンピュータにクローンします。たとえば、HTTPS 形式を使用して GitHub リポジトリをクローンするには、`git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY` を実行します。

6. リポジトリをクローンしたら、<Constant name="cloud" /> プロジェクトの <Constant name="cloud_cli" /> を [configure](/docs/cloud/configure-cloud-cli) します。これにより、[`dbt environment show`](/reference/commands/dbt-environment) などの dbt コマンドを実行して <Constant name="cloud" /> の設定を確認したり、`dbt compile` を実行してプロジェクトをコンパイルし、モデルとテストを検証したりできます。また、ファイルの追加、編集、リポジトリとの同期も可能です。

</TabItem>

<TabItem value="linux" label="Linux (native executable)">

ご使用のオペレーティング システムでパスの競合が発生した場合は、[FAQ](#faqs) を参照してください。

1. [GitHub](https://github.com/dbt-labs/dbt-cli/releases) から、ご使用のプラットフォーム向けの最新の Linux リリースをダウンロードします。(CPU アーキテクチャに基づいてファイルを選択してください)

2. `dbt-cloud-cli` バイナリを dbt プロジェクトと同じフォルダに解凍します。

  ```bash
  tar -xf dbt_0.29.9_linux_amd64.tar.gz
  ./dbt --version
  ```

:::info

上級ユーザーは、シェル プロファイルの PATH 環境変数に dbt CLI 実行可能ファイルを追加することで、複数のプロジェクトが同じ dbt CLI 実行可能ファイルを使用するように構成できます。

:::

3. コマンドラインで「./dbt --help」を実行してインストールを確認します。以下の出力が表示されれば、インストールは正しく行われています:

      ```bash
      The dbt CLI - an ELT tool for running SQL transformations and data models in dbt...
      ```

     この出力が表示されない場合は、pyenv または venv が無効化されていること、およびグローバル dbt バージョンがインストールされていないことを確認してください。

   * 環境の起動時に `dbt deps` コマンドを実行する必要がなくなったことに注意してください。この手順は以前は初期化時に必要でした。ただし、`packages.yml` ファイルに変更を加えた場合は、引き続き `dbt deps` を実行する必要があります。

4. `git clone` を使用して、リポジトリをローカルコンピュータにクローンします。たとえば、HTTPS 形式を使用して GitHub リポジトリをクローンするには、`git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY` を実行します。

5. リポジトリをクローンしたら、<Constant name="cloud" /> プロジェクトの <Constant name="cloud_cli" /> を [configure](/docs/cloud/configure-cloud-cli) します。これにより、[`dbt environment show`](/reference/commands/dbt-environment) などの dbt コマンドを実行して <Constant name="cloud" /> の設定を確認したり、`dbt compile` を実行してプロジェクトをコンパイルし、モデルとテストを検証したりできます。また、ファイルの追加、編集、リポジトリとの同期も可能です。

</TabItem>

<TabItem value="pip" label="Existing dbt Core users (pip)">

dbt Core が既にインストールされている場合、<Constant name="cloud_cli" /> が競合する可能性があります。以下の点にご注意ください。

- **競合を防ぐ** <br /> `pip` で <Constant name="cloud_cli" /> と <Constant name="core" /> の両方を使用し、新しい仮想環境を作成してください。<br /><br />
- **brew またはネイティブインストールで <Constant name="cloud_cli" /> と <Constant name="core" /> の両方を使用する** <br /> Homebrew を使用する場合は、競合を回避するために <Constant name="cloud_cli" /> に「dbt-cloud」というエイリアスを設定することを検討してください。詳細については、[FAQ](#faqs) をご覧ください。オペレーティング システムでパスの競合が発生した場合は、こちらをご覧ください。<br /><br />
- **<Constant name="cloud_cli" /> から dbt Core に戻す** <br />
   すでに <Constant name="cloud_cli" /> をインストールしていて、dbt Core に戻す必要がある場合:<br />
   - コマンド `pip uninstall dbt` を使用して、<Constant name="cloud_cli" /> をアンインストールします。
   - 次のコマンドを使用して、<Constant name="core" /> を再インストールします。「adapter_name」は適切なアダプタ名に置き換えてください。
    ```shell
    python -m pip install dbt-adapter_name --force-reinstall
    ```
    たとえば、Snowflakeをアダプタとして使用した場合、次を実行します: `python -m pip install dbt-snowflake --force-reinstall`

--------

<Constant name="cloud_cli" /> をインストールする前に、Python がインストールされ、仮想環境 venv または pyenv が設定されていることを確認してください。すでに Python 環境が設定されている場合は、[pip インストール手順](#install-dbt-cloud-cli-in-pip) に進んでください。

### 仮想環境をインストールする

仮想環境（venv）を使用して、名前空間「cloud-cli」を設定することをお勧めします。

1. 次のコマンドで、「dbt-cloud」という名前の新しい仮想環境を作成します。
   ```shell
   python3 -m venv dbt-cloud
    ```

2. シェルウィンドウまたはセッションを作成するたびに、仮想環境をアクティベートします。方法は、お使いのオペレーティングシステムによって異なります。

   - Mac および Linux の場合: `source dbt-cloud/bin/activate`<br/>
   - Windows の場合: `dbt-env\Scripts\activate`

3. (Mac および Linux のみ) 新しいシェルウィンドウまたはセッションごとに dbt 環境をアクティベートするためのエイリアスを作成します。シェルの設定ファイル (例: `$HOME/.bashrc、 $HOME/.zshrc`) に以下のコードを追加します。`<PATH_TO_VIRTUAL_ENV_CONFIG>` は、仮想環境の設定へのパスに置き換えてください:

   ```shell
   alias env_dbt='source <PATH_TO_VIRTUAL_ENV_CONFIG>/bin/activate'
   ```

### pip で dbt CLI をインストールします

1. (オプション) 既に <Constant name="core" /> がインストールされている場合は、このインストールによってそのパッケージが上書きされます。後で再インストールする必要がある場合に備えて、以下のコマンドを実行して <Constant name="core" /> のバージョンを確認してください。

  ```bash
 dbt --version
  ```

2. 仮想環境にいることを確認し、次のコマンドを実行して <Constant name="cloud" /> CLI をインストールします。

  ```bash
  pip install dbt --no-cache-dir
  ```

  インストールに問題がある場合は、`--force-reinstall` 引数を指定してコマンドを実行すると解決する可能性があります:

   ```bash
   pip install dbt --no-cache-dir --force-reinstall
   ``` 

3. （オプション）<Constant name="core" /> に戻すには、まず <Constant name="cloud" /> CLI と <Constant name="core" /> の両方をアンインストールします。その後、<Constant name="core" /> を再インストールします。

  ```bash
  pip uninstall dbt-core dbt
  pip install dbt-adapter_name --force-reinstall
  ```

4. `git clone` を使用して、リポジトリをローカルコンピュータにクローンします。たとえば、HTTPS 形式を使用して GitHub リポジトリをクローンするには、`git clone https://github.com/YOUR-USERNAME/YOUR-REPOSITORY` を実行します。

5. リポジトリをクローンしたら、<Constant name="cloud" /> プロジェクトの <Constant name="cloud_cli" /> を [configure](/docs/cloud/configure-cloud-cli) します。これにより、[`dbt environment show`](/reference/commands/dbt-environment) を使用して <Constant name="cloud" /> 構成を表示したり、`dbt compile` を使用してプロジェクトをコンパイルし、モデルとテストを検証したりといった dbt コマンドを実行できるようになります。また、リポジトリにファイルを追加、編集、同期することもできます。

</TabItem>


</Tabs>

## dbt CLI を更新する

以下の手順では、お使いのオペレーティング システムに応じて、<Constant name="cloud_cli" /> を最新バージョンに更新する方法について説明します。


<Tabs>

<TabItem value="mac" label="macOS (brew)">

<Constant name="cloud_cli" /> を更新するには、`brew update` を実行してから `brew upgrade dbt` を実行します。

</TabItem>

<TabItem value="windows" label="Windows (executable)">

更新するには、[Windows](/docs/cloud/cloud-cli-installation?install=windows#install-dbt-cloud-cli) で説明されているのと同じプロセスに従い、既存の `dbt.exe` 実行可能ファイルを新しいものに置き換えます。

</TabItem>

<TabItem value="linux" label="Linux (executable)">

更新するには、[Linux](/docs/cloud/cloud-cli-installation?install=linux#install-dbt-cloud-cli) で説明されているのと同じプロセスに従い、既存の `dbt` 実行可能ファイルを新しいものに置き換えます。

</TabItem>

<TabItem value="existing" label="Existing dbt Core users (pip)">

アップデートするには：
- 仮想環境にいることを確認してください。
- `python -m pip install --upgrade dbt` を実行してください。
	
</TabItem>

</Tabs>
  
  
## Considerations

import CloudCliRelativePath from '/snippets.ja/_cloud-cli-relative-path.md';

<CloudCliRelativePath />

## FAQs

<DetailsToggle alt_header="dbt CLI と dbt Core の違いは何ですか?">

<Constant name="cloud_cli" /> とオープンソース プロジェクトである <a href="https://github.com/dbt-labs/dbt-core">dbt Core</a> はどちらも、dbt コマンドを実行できるコマンドライン ツールです。

主な違いは、<Constant name="cloud_cli" /> は <Constant name="cloud" /> のインフラストラクチャに合わせてカスタマイズされており、そのすべての <a href="https://docs.getdbt.com/docs/cloud/about-cloud/dbt-cloud-features">機能</a> と統合されていることです。

</DetailsToggle>

<DetailsToggle alt_header="dbt CLI と dbt Core の両方を実行するにはどうすればよいですか?">

互換性のため、`dbt` を実行すると <Constant name="cloud_cli" /> と <Constant name="core" /> の両方が呼び出されます。これにより、オペレーティング システムが $PATH 環境変数（設定）に基づいてどちらか一方を選択した場合、パスの競合が発生する可能性があります。

<Constant name="core" /> がローカルにインストールされている場合は、次のいずれかを実行してください。

1. <code>pip3 install dbt</code> [pip](/docs/cloud/cloud-cli-installation?install=pip#install-dbt-cloud-cli) コマンドを使用してインストールします。
2. ネイティブインストールを実行し、<Constant name="core" /> を含む仮想環境を無効化するか、<Constant name="cloud_cli" /> のエイリアスを作成します。
3. (上級ユーザー向け) ネイティブインストールを行いますが、$PATH 環境変数を <Constant name="cloud_cli" /> バイナリを正しく指定するように変更し、<Constant name="cloud_cli" /> と <Constant name="core" /> の両方を併用します。

<Constant name="cloud_cli" /> をアンインストールすれば、いつでも <Constant name="core" /> を再び使用できます。

</DetailsToggle>

<DetailsToggle alt_header="エイリアスを作成するにはどうすればいいですか?">

<Constant name="cloud_cli" /> のエイリアスを作成するには: <br />

1. シェルのプロファイル設定ファイルを開きます。シェルとシステムに応じて、`~/.bashrc`、`~/.bash_profile`、`~/.zshrc` などのファイルになります。<br />

2. <Constant name="cloud_cli" /> バイナリを指すエイリアスを追加します。例: <code>alias dbt-cloud="path_to_dbt_cloud_cli_binary</code>

   <code>path_to_dbt_cloud_cli_binary</code> を、<Constant name="cloud_cli" /> バイナリへの実際のパス (<code>/opt/homebrew/bin/dbt</code>) に置き換えます。このエイリアスを使用すると、コマンド <code>dbt-cloud</code> を使用して <Constant name="cloud_cli" /> を呼び出すことができます。<br />

3. ファイルを保存し、シェルを再起動するか、プロファイルファイルに対して <code>source</code> を実行して変更を適用します。
例えば、bash の場合は次のように実行します: <code>source ~/.bashrc</code><br />

4. エイリアスを使用してコマンドを実行し、テストします。<br />
   - <Constant name="cloud_cli" /> を実行するには、<code>dbt-cloud</code> コマンドを使用します。 <code>dbt-cloud command_name</code>。「command_name」は、実行する特定のdbtコマンドに置き換えてください。<br />
   - dbt Coreを実行するには、<code>dbt</code>コマンドを使用します：<code>dbt command_name</code>。「command_name」は、実行する特定のdbtコマンドに置き換えてください。<br />


このエイリアスを使用すると、dbt Core がネイティブにインストールされているときに、<code>dbt-cloud</code> コマンドを使用して <Constant name="cloud_cli" /> を呼び出すことができます。

</DetailsToggle>

<DetailsToggle alt_header="新しいコマンドを実行しようとすると、「Stuck session」エラーが表示されるのはなぜですか?">

<Constant name="cloud_cli" /> では、データウェアハウスへの書き込みコマンドは一度に 1 つだけ実行できます。複数の書き込みコマンド（例: `dbt run` と `dbt build`）を同時に実行しようとすると、`stuck session` エラーが発生します。この問題を解決するには、特定の呼び出しの ID を cancel コマンドに渡してキャンセルしてください。詳細については、[並列実行](/reference/dbt-commands#parallel-execution) を参照してください。

</DetailsToggle>

<FAQ path="Troubleshooting/long-sessions-cloud-cli" />
  
