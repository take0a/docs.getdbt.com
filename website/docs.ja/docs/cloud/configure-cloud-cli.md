---
title: Configure and use the dbt CLI
id: configure-cloud-cli
description: "Instructions on how to configure the dbt CLI"
sidebar_label: "Configuration and usage"
pagination_next: null
---

<Constant name="cloud" /> プロジェクトの <Constant name="cloud_cli" /> を構成して、`dbt environment show` で <Constant name="cloud" /> の構成を表示したり、`dbt compile` でプロジェクトをコンパイルしてモデルとテストを検証したりするなど、dbt コマンドを実行する方法を学びます。また、以下のメリットも得られます。

- <Constant name="cloud" /> プラットフォームにおける安全な認証情報ストレージ。
- ビルド成果物の Cloud プロジェクトの本番環境への [自動延期](/docs/cloud/about-cloud-develop-defer)。
- より高速で低コストのビルド。
- <Constant name="mesh" /> のサポート ([プロジェクト間参照](/docs/mesh/govern/project-dependencies))、その他。

## 前提条件

- <Constant name="cloud" /> でプロジェクトをセットアップする必要があります。
  - **注** &mdash; <Constant name="cloud_cli" /> を使用している場合は、<Constant name="cloud" /> インターフェースから直接 [データプラットフォーム](/docs/cloud/connect-data-platform/about-connections) に接続できるため、[`profiles.yml`](/docs/core/connect-data-platform/profiles.yml) ファイルは必要ありません。
- そのプロジェクトに [個人開発認証情報](/docs/dbt-cloud-environments#set-developer-credentials) が設定されている必要があります。<Constant name="cloud" /> CLI は、<Constant name="cloud" /> に安全に保存されているこれらの認証情報を使用して、データプラットフォームと通信します。
- dbt バージョン 1.5 以上を使用する必要があります。アップグレードするには、[<Constant name="cloud" /> バージョン](/docs/dbt-versions/upgrade-dbt-version-in-cloud) を参照してください。

## dbt CLI を構成する

<Constant name="cloud_cli" /> をインストールしたら、<Constant name="cloud" /> プロジェクトに接続できるように構成する必要があります。

1. <Constant name="cloud" /> で **Develop** に移動し、**Configure <Constant name="cloud_cli" />** をクリックして `dbt_cloud.yml` 認証情報ファイルをダウンロードします。

    <details>
    <summary>資格情報をダウンロードするための地域の URL</summary>
    地域に応じて提供されるリンクから資格情報をダウンロードすることもできます:

    - North America: <a href="https://cloud.getdbt.com/cloud-cli">https://cloud.getdbt.com/cloud-cli</a>
    - EMEA: <a herf="https://emea.dbt.com/cloud-cli">https://emea.dbt.com/cloud-cli</a>
    - APAC: <a href="https://au.dbt.com/cloud-cli">https://au.dbt.com/cloud-cli</a>
    - North American Cell 1: <code>https:/ACCOUNT_PREFIX.us1.dbt.com/cloud-cli</code>
    - Single-tenant: <code>https://YOUR_ACCESS_URL/cloud-cli</code>

    </details>

2. `dbt_cloud.yml` ファイルを、<Constant name="cloud_cli" /> の設定が保存されている `.dbt` ディレクトリに保存してください。API キーが含まれているため、安全な場所に保管してください。`.dbt` ディレクトリの作成方法と `dbt_cloud.yml` ファイルの移行方法については、[FAQ](#faqs) をご覧ください。
   
    - North America: https://YOUR_ACCESS_URL/cloud-cli
    - EMEA: https://emea.dbt.com/cloud-cli
    - APAC: https://au.dbt.com/cloud-cli
    - North American Cell 1: `https:/ACCOUNT_PREFIX.us1.dbt.com/cloud-cli`
    - Single-tenant: `https://YOUR_ACCESS_URL/cloud-cli`
  
3. バナーの指示に従って、次の場所に構成ファイルをダウンロードします:
   - Mac or Linux:  `~/.dbt/dbt_cloud.yml`
   - Windows:  `C:\Users\yourusername\.dbt\dbt_cloud.yml`  

  設定ファイルは次のようになります:

  ```yaml
  version: "1"
  context:
    active-project: "<project id from the list below>"
    active-host: "<active host from the list>"
    defer-env-id: "<optional defer environment id>"
  projects:
    - project-name: "<project-name>"
      project-id: "<project-id>"
      account-name: "<account-name>"
      account-id: "<account-id>"
      account-host: "<account-host>" # for example, "cloud.getdbt.com"
      token-name: "<pat-or-service-token-name>"
      token-value: "<pat-or-service-token-value>"
  
    - project-name: "<project-name>"
      project-id: "<project-id>"
      account-name: "<account-name>"
      account-id: "<account-id>"
      account-host: "<account-host>" # for example, "cloud.getdbt.com"
      token-name: "<pat-or-service-token-name>"
      token-value: "<pat-or-service-token-value>"  
  ```

1. 設定ファイルをダウンロードしてディレクトリを作成したら、ターミナルでプロジェクトに移動します:

    ```bash
    cd ~/dbt-projects/jaffle_shop
    ```

2. `dbt_project.yml` ファイルに、`project-id` フィールドを含む `dbt-cloud` セクションが存在するか、または含めていることを確認してください。`project-id` フィールドには、使用する <Constant name="cloud" /> プロジェクト ID を指定します。

    ```yaml
    # dbt_project.yml
    name:
    version:
    # Your project configs...

    dbt-cloud: 
        project-id: PROJECT_ID
    ```

   - プロジェクトIDを確認するには、<Constant name="cloud" /> ナビゲーションメニューで **Develop** を選択してください。プロジェクトIDはURLで確認できます。例えば、`https://YOUR_ACCESS_URL/develop/26228/projects/123456` の場合、プロジェクトIDは `123456` です。

3. これで、[<Constant name="cloud_cli" /> を使用](#use-the-dbt-cloud-cli)し、[`dbt environment show`](/reference/commands/dbt-environment)などの[dbtコマンド](/reference/dbt-commands)を実行して<Constant name="cloud" />の設定詳細を表示したり、`dbt compile` を使用してdbtプロジェクト内のモデルをコンパイルしたりできるようになります。

リポジトリを再クローンすると、ファイルの追加、編集、リポジトリとの同期が可能になります。

## 環境変数を設定する

<Constant name="cloud" /> CLI で dbt プロジェクトの環境変数を設定するには、次の手順に従います。

1. <Constant name="cloud" /> の左側のメニューでアカウント名をクリックし、**Account settings** を選択します。
2. **Your profile** セクションで、**Credentials** を選択します。
3. プロジェクトをクリックし、**Environment variables** セクションまでスクロールします。
4. 右下の **Edit** をクリックし、ユーザーレベルの環境変数を設定します。

## dbt CLI を使用する

<Constant name="cloud_cli" /> は、dbt Core と同じ [dbt コマンド](/reference/dbt-commands) と [MetricFlow コマンド](/docs/build/metricflow-commands) のセットを使用して、指定したコマンドを実行します。たとえば、[`dbt environment`](/reference/commands/dbt-environment) コマンドを使用して、<Constant name="cloud" /> の設定の詳細を確認できます。<Constant name="cloud_cli" /> を使用すると、次のことが可能になります。

- [複数の呼び出しを並列実行](/reference/dbt-commands) し、[安全な並列処理](/reference/dbt-commands#parallel-execution) を確保します。これは現在、`dbt-core` では保証されていません。
- ビルド成果物を Cloud プロジェクトの運用環境に自動的に延期します。
- [プロジェクト依存関係](/docs/mesh/govern/project-dependencies)をサポートしており、<Constant name="cloud" /> のメタデータサービスを使用して別のプロジェクトに依存できます。
  - プロジェクト依存関係は、他のプロジェクトで定義されたパブリックモデルに即座に接続して参照（または `ref`）します。これらの上流モデルを自分で実行したり分析したりする必要はありません。代わりに、データセットを返すAPIとして扱います。
 
:::tip Use the <code>--help</code> flag
ヒントとして、ほとんどのコマンドラインツールには、使用可能なコマンドと引数を表示するための `--help` フラグがあります。dbt では `--help` フラグを次の2つの方法で使用できます。
- `dbt --help`: dbt で使用可能なコマンドを一覧表示します<br />
- `dbt run --help`: `run` コマンドで使用可能なフラグを一覧表示します
:::
 
## SQL ファイルの lint 処理

<Constant name="cloud" /> CLI から、[SQLFluff](https://sqlfluff.com/) を呼び出すことができます。これは、複雑な関数、構文、フォーマット、コンパイルエラーを警告する、モジュール式で設定可能な SQL リンターです。SQLFluff に渡すことができるフラグの多くは、<Constant name="cloud" /> CLI でも使用できます。

使用可能な SQLFluff コマンドは次のとおりです。

- `lint` - ファイルリストまたは標準入力 (stdin) を渡して SQL ファイルを lint 処理します。
- `fix` - SQL ファイルを修正します。
- `format` - SQL ファイルを自動フォーマットします。

SQL ファイルを lint 処理するには、次のコマンドを実行します。

```
dbt sqlfluff lint [PATHS]... [flags]
```

パスが設定されていない場合、dbt は現在のプロジェクト内のすべての SQL ファイルを lint します。特定の SQL ファイルまたはディレクトリを lint するには、`PATHS` を SQL ファイルまたはファイルのディレクトリのパスに設定します。複数のファイルまたはディレクトリを lint するには、複数の `PATHS` フラグを渡します。

dbt でサポートされているすべてのコマンドとフラグの詳細情報を表示するには、`dbt sqlfluff -h` コマンドを実行します。

#### 考慮事項

<Constant name="cloud_cli" /> から `dbt sqlfluff` を実行する場合、以下の重要な動作について考慮する必要があります。

- dbt は、`.sqlfluff` ファイルが存在する場合、カスタム構成があるかどうか確認するためにそれを読み取ります。
- 継続的インテグレーション/継続的開発 (CI/CD) ワークフローの場合、プロジェクトに `dbt_cloud.yml` ファイルが存在し、この dbt プロジェクト内からコマンドを正常に実行している必要があります。
- SQLFluff コマンドは、ファイル違反が発生した場合、終了コード 0 を返します。この dbt の動作は、リンティング違反で 0 以外の終了コードが返される SQLFluff の動作とは異なります。dbt Labs は、今後のリリースでこの問題に対処する予定です。

## 考慮事項

import CloudCliRelativePath from '/snippets.ja/_cloud-cli-relative-path.md';

<CloudCliRelativePath />

## FAQs

<DetailsToggle alt_header=".dbtディレクトリを作成してファイルを移動する方法">

`.dbt` ディレクトリがまだない場合は、以下の推奨手順に従って作成してください。既に `.dbt` ディレクトリがある場合は、`dbt_cloud.yml` ファイルをそこに移動します。

<Tabs>
<TabItem value="Create a .dbt directory">

  1. dbt プロジェクト リポジトリをローカルにクローンします。
  2. `mkdir` コマンドに続けて作成したいフォルダ名を指定します。`~` プレフィックスを追加して、ファイルシステムのルートに `.dbt` フォルダを作成します。

     ```bash
     mkdir ~/.dbt
     ```

これにより、ルートディレクトリに「.dbt」フォルダが作成されます。

Macユーザーの場合、このフォルダは隠しフォルダ（ドットプレフィックスのため）であるため、デフォルトではFinderに表示されません。隠しファイルと隠しフォルダを表示するには、Command + Shift + Gを押してください。

</TabItem>

<TabItem value="Move the dbt_cloud.yml file">

### Mac または Linux
コマンドラインで `mv` コマンドを使用して、`dbt_cloud.yml` ファイルを `.dbt` ディレクトリに移動します。`dbt_cloud.yml` ファイルをダウンロードしてダウンロードフォルダに保存している場合は、コマンドは次のようになります。

```bash
mv ~/Downloads/dbt_cloud.yml ~/.dbt/dbt_cloud.yml
```

### Windows
コマンドラインでmoveコマンドを使用します。ファイルがダウンロードフォルダにあると仮定すると、コマンドは次のようになります。

```bash
move %USERPROFILE%\Downloads\dbt_cloud.yml %USERPROFILE%\.dbt\dbt_cloud.yml
```

</TabItem>
</Tabs>

このコマンドは、`dbt_cloud.yml` を `Downloads` フォルダから `.dbt` フォルダに移動します。`dbt_cloud.yml` ファイルが他の場所にある場合は、パスを調整してください。

</DetailsToggle>

<DetailsToggle alt_header="アーティファクトのダウンロードをスキップする方法">

デフォルトでは、<Constant name="cloud_cli" /> から dbt コマンドを実行すると、[すべてのアーティファクト](/reference/artifacts/dbt-artifacts) がダウンロードされます。これらのファイルのダウンロードをスキップするには、実行するコマンドに `--download-artifacts=false` を追加します。これにより実行時のパフォーマンスが向上しますが、[マニフェスト](/reference/artifacts/manifest-json) などのアセットに依存するワークフローが中断される可能性があります。

</DetailsToggle>

<FAQ path="Troubleshooting/long-sessions-cloud-cli" />
