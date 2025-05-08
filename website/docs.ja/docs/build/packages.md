---
title: "パッケージ"
id: "packages"
description:  "dbt パッケージは、コードをモジュール化し、データを効率的に変換するのに役立ちます。"
keywords: [dbt package, private package, dbt private package, dbt data transformation, dbt clone, add dbt package]
---


ソフトウェアエンジニアは、コードをライブラリとしてモジュール化することがよくあります。これらのライブラリは、プログラマーが独自のビジネスロジックに集中する時間を増やし、誰かが既に時間をかけて完成させたコードの実装に費やす時間を減らすのに役立ちます。

dbt では、このようなライブラリは「パッケージ」と呼ばれます。dbt のパッケージが非常に強力なのは、私たちが直面した分析上の問題の多くが組織間で共有されているためです。例えば、次のようなことが挙げられます。
* 一貫性のある構造を持つ SaaS データセットからのデータの変換。例:
  * [Snowplow](https://hub.getdbt.com/dbt-labs/snowplow/latest/) または [Segment](https://hub.getdbt.com/dbt-labs/segment/latest/) のページビューをセッションに変換する。
  * [AdWords](https://hub.getdbt.com/dbt-labs/adwords/latest/) または [Facebook Ads](https://hub.getdbt.com/dbt-labs/facebook_ads/latest/) の支出データを一貫した形式に変換する。
* 同様の機能を実行する dbt マクロの作成。例:
  * [SQL の生成](https://github.com/dbt-labs/dbt-utils#sql-helpers) による 2 つのリレーションの結合、列のピボット、または <Term id="surrogate-key" /> の構築
  * [カスタム スキーマ テスト](https://github.com/dbt-labs/dbt-utils#schema-tests) の作成
  * [監査クエリ](https://hub.getdbt.com/dbt-labs/audit_helper/latest/) の作成
* データスタックで使用される特定のツール用のモデルとマクロの構築。例:
  * [Redshift](https://hub.getdbt.com/dbt-labs/redshift/latest/) の権限を理解するためのモデル
  * [Stitch](https://hub.getdbt.com/dbt-labs/stitch_utils/latest/) によってロードされたデータを操作するマクロ

dbt パッケージは、実際にはスタンドアロンの dbt プロジェクトであり、特定の問題領域に対処するモデル、マクロ、その他のリソースが含まれています。dbt ユーザーがプロジェクトにパッケージを追加すると、そのパッケージのすべてのリソースが自分のプロジェクトの一部になります。つまり、次のようになります。
* パッケージ内のモデルは、`dbt run` を実行すると実体化されます。
* 自分のモデルで `ref` を使用して、パッケージのモデルを参照できます。
* `source` を使用して、パッケージ内のソースを参照できます。
* パッケージ内のマクロを自分のプロジェクトで使用できます。
* dbt パッケージの定義とインストールは、[Python パッケージの定義とインストール](/docs/build/python-models#using-pypi-packages) とは異なることに注意してください。

import UseCaseInfo from '/snippets.ja/_packages_or_dependencies.md';

<UseCaseInfo/>

## プロジェクトにパッケージを追加するにはどうすればよいですか？
1. `dependencies.yml` または `packages.yml` という名前のファイルを dbt プロジェクトに追加します。これは `dbt_project.yml` ファイルと同じ階層に配置する必要があります。
2. サポートされている構文のいずれかを使用して、追加するパッケージを指定します。例:

<File>

```yaml
packages:
  - package: dbt-labs/snowplow
    version: 0.7.0

  - git: "https://github.com/dbt-labs/dbt-utils.git"
    revision: 0.9.2

  - local: /opt/dbt/redshift
```

</File>

デフォルトの [`packages-install-path`](/reference/project-configs/packages-install-path) は `dbt_packages` です。

3. `dbt deps` を実行してパッケージをインストールします。パッケージは `dbt_packages` ディレクトリにインストールされます。デフォルトでは、パッケージのソースコードの重複を避けるため、このディレクトリは Git によって無視されます。

## パッケージを指定するにはどうすればよいですか？
パッケージが保存されている場所に応じて、次のいずれかの方法でパッケージを指定できます。

### ハブパッケージ（推奨）

dbt Labs は、dbt コミュニティへのサービスとして、dbt パッケージのレジストリである [パッケージハブ](https://hub.getdbt.com) をホストしていますが、パッケージの整合性、操作性、有効性、セキュリティについて保証または確認するものではありません。ハブパッケージをインストールする前に、[dbt Labs パッケージ免責事項](https://hub.getdbt.com/disclaimer/) をお読みください。

利用可能なハブパッケージは、以下の方法でインストールできます:

<File name='packages.yml'>

```yaml
packages:
  - package: dbt-labs/snowplow
    version: 0.7.3 # version number
```

</File>

Hubパッケージではバージョンを指定する必要があります。最新のリリース番号はdbt Hubで確認できます。Hubパッケージは[セマンティックバージョニング](https://semver.org/)を使用しているため、パッケージを特定のマイナーリリースの最新パッチバージョンに固定することを推奨します。例：


```yaml
packages:
  - package: dbt-labs/snowplow
    version: [">=0.7.0", "<0.8.0"]
```

`dbt deps` はデフォルトで各パッケージを「ピン留め」します。詳細については、[「パッケージのピン留め」](#pinning-packages) をご覧ください。

可能な場合は、dbt Hub 経由でパッケージをインストールすることをお勧めします。これにより、dbt は重複する依存関係を処理できます。これは、次のような状況で役立ちます。
* プロジェクトで dbt-utils パッケージと Snowplow パッケージの両方を使用しており、Snowplow パッケージも dbt-utils パッケージを使用している場合。
* プロジェクトで Snowplow パッケージと Stripe パッケージの両方を使用しており、どちらも dbt-utils パッケージを使用している場合。

一方、他のパッケージインストール方法では、重複する dbt-utils パッケージを処理できません。

上級ユーザーは、[このリポジトリ](https://github.com/dbt-labs/hub.getdbt.com) と `DBT_PACKAGE_HUB_URL` 環境変数の設定に基づいて、パッケージハブの内部バージョンをホストすることもできます。

#### プレリリース版

パッケージメンテナーの中には、新機能や新しいバージョンのdbtとの互換性をテストするために、プレリリース版のパッケージをdbt Hubにプッシュしたいと考える方がいます。プレリリース版は、`a1`（最初のアルファ版）、`b2`（2番目のベータ版）、`rc3`（3番目のリリース候補版）などのサフィックスで区別されます。

デフォルトでは、`dbt deps`はパッケージの依存関係を解決する際にプレリリース版を含めません。プレリリースのインストールを有効にするには、以下の2つの方法があります。
- `version` 条件でプレリリースバージョンを明示的に指定する
- `install_prerelease` を `true` に設定し、互換性のあるバージョン範囲を指定する

例えば、以下のどちらの構成でも、[`dbt_artifacts` パッケージ](https://hub.getdbt.com/brooklyn-data/dbt_artifacts/latest/) の `0.4.5-a2` が正常にインストールされます。

```yaml
packages:
  - package: brooklyn-data/dbt_artifacts
    version: 0.4.5-a2
```

```yaml
packages:
  - package: brooklyn-data/dbt_artifacts
    version: [">=0.4.4", "<0.4.6"]
    install_prerelease: true
```

### Git パッケージ
Git サーバーに保存されたパッケージは、次のように `git` 構文を使用してインストールできます:

<File name='packages.yml'>

```yaml
packages:
  - git: "https://github.com/dbt-labs/dbt-utils.git" # git URL
    revision: 0.9.2 # tag or branch name
```

</File>

パッケージの Git URL を追加し、必要に応じてリビジョンを指定します。リビジョンは次のいずれかになります。
- ブランチ名
- タグ付きリリース
- 特定のコミット（40文字のハッシュ全体）

40文字のハッシュを指定したリビジョンの例：

```yaml
packages:
  - git: "https://github.com/dbt-labs/dbt-utils.git"
    revision: 4e28d6da126e2940d17f697de783a717f2503188
```

デフォルトでは、`dbt deps` は各パッケージを「ピン留め」します。詳細については、[「パッケージのピン留め」](#pinning-packages) を参照してください。

### 内部でホストされている tarball URL

組織によっては、セキュリティ要件によりリソースを内部サービスからのみプルする必要がある場合があります。Artifactory やクラウドストレージバケットなどのホスト環境からパッケージをインストールする必要性に対応するため、dbt Core では内部でホストされている tarball URL からパッケージをインストールできます。 


```yaml
packages:
  - tarball: https://codeload.github.com/dbt-labs/dbt-utils/tar.gz/0.9.6
    name: 'dbt_utils'
```

ここで、`name: 'dbt_utils'` は、パッケージ ソース コードをインストールするために作成された `dbt_packages` のサブフォルダーを指定します。

## プライベートパッケージ

### ネイティブプライベートパッケージ <Lifecycle status='beta'/> 

dbt Cloud は、環境内の既存の [構成](/docs/cloud/git/git-configuration-in-dbt-cloud) を活用して、[サポート対象](#前提条件) Git リポジトリからのプライベートパッケージをサポートします。以前は、プライベートリポジトリからパッケージを取得するには [トークン](#git-token-method) を設定する必要がありました。

#### 前提条件

- ネイティブのプライベートパッケージを使用するには、**Account settings** の **Integrations** セクションで、以下のいずれかの Git プロバイダーが設定されている必要があります。
  - [GitHub](/docs/cloud/git/connect-github)
  - [Azure DevOps](/docs/cloud/git/connect-azure-devops)
    - プライベートパッケージは、単一の Azure DevOps プロジェクト内でのみ機能します。リポジトリが同じ組織内の異なるプロジェクトにある場合、現時点では `private` キーでそれらを参照することはできません。
    - Azure DevOps の場合は、統合ソースリポジトリから継承されたプロジェクト層を使用して、`org/repo` パス（`org_name/project_name/repo_name` パスではありません）を使用してください。
  - GitLab のサポートは近日中に開始されます。

#### 構成

`packages.yml` または `dependencies.yml` 内の `private` キーを使用すると、アクセストークンをプロビジョニングしたり、dbt Cloud 環境変数を作成したりすることなく、既存の dbt Cloud Git 統合を使用してパッケージリポジトリをクローンできます。


<File name="packages.yml">

```yaml
packages:
  - private: dbt-labs/awesome_repo # your-org/your-repo path
  - package: normal packages
  [...]
```
</File>

:::tip Azure DevOps に関する考慮事項

- 現在、プライベートパッケージは、パッケージリポジトリがソースリポジトリと同じ Azure DevOps プロジェクト内にある場合にのみ機能します。
- `private` キーには、通常の ADO の `org_name/project_name/repo_name` パスではなく、`org/repo` パスを使用してください。
- 異なる Azure DevOps プロジェクト内のリポジトリは、将来のアップデートまでサポートされません。

`private` キーに `org/repo` を指定することで、プライベートパッケージを使用できます。

<File name="packages.yml">

```yaml
packages:
  - private: my-org/my-repo # Works if your ADO source repo and package repo are in the same project
```
</File>
:::

通常の dbt パッケージと同様に、プライベート パッケージをピン留めできます:

```yaml
packages:
  - private: dbt-labs/awesome_repo
    revision: "0.9.5" # Pin to a tag, branch, or complete 40-character commit hash
  
```

複数の Git 統合を使用している場合は、プロバイダー キーを追加して曖昧さを解消します:

```yaml
packages:
  - private: dbt-labs/awesome_repo
    provider: "github" # GitHub and Azure are currently supported. GitLab is coming soon.

```

この方法を使用すると、接続するための追加の手順なしで、統合された Git プロバイダーからプライベート パッケージを取得できます。

### SSHキー方式（コマンドラインのみ）
コマンドラインを使用している場合、プライベートパッケージはSSHとSSHキーを介してクローンできます。

SSHキーを使用してgitリモートサーバーに認証すると、ユーザー名とパスワードを毎回入力する必要がなくなります。SSHキー、その生成方法、およびgitプロバイダーへの追加方法の詳細については、[Github](https://docs.github.com/en/github/authenticating-to-github/connecting-to-github-with-ssh) および [GitLab](https://docs.gitlab.com/ee/user/ssh.html) をご覧ください。


<File name='packages.yml'>

```yaml
packages:
  - git: "git@github.com:dbt-labs/dbt-utils.git" # git SSH URL
```

</File>

dbt Cloud を使用している場合、SSH キー メソッドは機能しませんが、[HTTPS Git トークン メソッド](https://docs.getdbt.com/docs/build/packages#git-token-method) を使用できます。


### Gitトークンメソッド {#git-token-method}

:::note

dbt Cloud は、GitHub および Azure DevOps（GitLab は近日提供開始）で Git ホストされたプライベートパッケージを[ネイティブサポート](#native-private-packages)します。サポートされている [統合 Git 環境](/docs/cloud/git/git-configuration-in-dbt-cloud) をご利用の場合は、プライベートパッケージを取得するために Git トークンを設定する必要がなくなりました。

:::

この方法では、ユーザーは環境変数を介してgitトークンを渡すことで、HTTPS経由でクローンを実行できます。使用するトークンの有効期限には注意してください。有効期限が切れていると、スケジュールされた実行が失敗する可能性があります。また、ユーザートークンは、ユーザーが特定のリポジトリへのアクセスを失った場合に問題を引き起こす可能性があります。


:::info dbt Cloud の使用
dbt Cloud を使用する場合は、環境変数の命名規則に従う必要があります。dbt Cloud の環境変数には、`DBT_` または `DBT_ENV_SECRET` のいずれかのプレフィックスを付ける必要があります。環境変数のキーは大文字で、大文字と小文字が区別されます。プロジェクトのコードで `{{env_var('DBT_KEY')}}` を参照する場合、キーは dbt Cloud の UI で定義された変数と完全に一致する必要があります。
:::

In GitHub:

<File name='packages.yml'>

```yaml
packages:
  # use this format when accessing your repository via a github application token
  - git: "https://{{env_var('DBT_ENV_SECRET_GIT_CREDENTIAL')}}@github.com/dbt-labs/awesome_repo.git" # git HTTPS URL

  # use this format when accessing your repository via a classical personal access token
  - git: "https://{{env_var('DBT_ENV_SECRET_GIT_CREDENTIAL')}}@github.com/dbt-labs/awesome_repo.git" # git HTTPS URL
 
   # use this format when accessing your repository via a fine-grained personal access token (username sometimes required)
  - git: "https://GITHUB_USERNAME:{{env_var('DBT_ENV_SECRET_GIT_CREDENTIAL')}}@github.com/dbt-labs/awesome_repo.git" # git HTTPS URL
```

</File>

GitHub Personal Access Tokenの作成について詳しくは、[こちら](https://docs.github.com/ja/enterprise-server@3.1/github/authenticating-to-github/keeping-your-account-and-data-secure/creating-a-personal-access-token)をご覧ください。GitHub Appインストール[トークン](https://docs.github.com/ja/rest/reference/apps#create-an-installation-access-token-for-an-app)も使用できます。

In GitLab:

<File name='packages.yml'>

```yaml
packages:
  - git: "https://{{env_var('DBT_USER_NAME')}}:{{env_var('DBT_ENV_SECRET_DEPLOY_TOKEN')}}@gitlab.example.com/dbt-labs/awesome_project.git" # git HTTPS URL
```

</File>

GitLab デプロイトークンの作成方法の詳細については[こちら](https://docs.gitlab.com/ee/user/project/deploy_tokens/#creating-a-deploy-token)、HTTPS URLの適切な構築方法については[こちら](https://docs.gitlab.com/ee/user/project/deploy_tokens/#git-clone-a-repository)をご覧ください。デプロイトークンはメンテナーのみが管理できます。

In Azure DevOps:

<File name='packages.yml'>

```yaml
packages:
  - git: "https://{{env_var('DBT_ENV_SECRET_PERSONAL_ACCESS_TOKEN')}}@dev.azure.com/dbt-labs/awesome_project/_git/awesome_repo" # git HTTPS URL
```

</File>

個人アクセス トークンの作成の詳細については、[こちら](https://docs.microsoft.com/en-us/azure/devops/organizations/accounts/use-personal-access-tokens-to-authenticate?view=azure-devops&tabs=preview-page#create-a-pat) を参照してください。

In Bitbucket:

<File name='packages.yml'>

```yaml
packages:
  - git: "https://{{env_var('DBT_USER_NAME')}}:{{env_var('DBT_ENV_SECRET_PERSONAL_ACCESS_TOKEN')}}@bitbucketserver.com/scm/awesome_project/awesome_repo.git" # for Bitbucket Server
```

</File>

個人アクセス トークンの作成の詳細については、[こちら](https://confluence.atlassian.com/bitbucketserver/personal-access-tokens-939515499.html) を参照してください。



## パッケージ化されたプロジェクトのサブディレクトリの設定

通常、dbt は `dbt_project.yml` がパッケージの最上位ファイルとして配置されていることを想定しています。パッケージ化されたプロジェクトがサブディレクトリ（たとえば、はるかに大きなモノリポジトリ内）にネストされている場合は、オプションでフォルダパスを `subdirectory` として指定できます。dbt は、そのサブディレクトリ内にあるファイルのみの[スパースチェックアウト](https://git-scm.com/docs/git-sparse-checkout) を試みます。最新バージョンの `git` (`>=2.26.0`) を使用する必要があります。

<File name='packages.yml'>

```yaml
packages:
  - git: "https://github.com/dbt-labs/dbt-labs-experimental-features" # git URL
    subdirectory: "materialized-views" # name of subdirectory containing `dbt_project.yml`
```

</File>

### ローカルパッケージ
「ローカル」パッケージとは、ローカルファイルシステムからアクセス可能な dbt プロジェクトです。プロジェクトのパスを指定してインストールできます。現在のプロジェクトのディレクトリを基準としたサブディレクトリ内にプロジェクトをネストすると、最も効果的に機能します。

<File name='packages.yml'>

```yaml
packages:
  - local: relative/path/to/subdirectory
```

</File>

他のパターンも場合によっては機能しますが、常に機能するとは限りません。例えば、このプロジェクトをパッケージとして別の場所にインストールしたり、別のシステムで実行したりする場合、相対パスと絶対パスは同じ結果になります。

<File name='packages.yml'>

```yaml
packages:
  # not recommended - support for these patterns vary
  - local: /../../redshift   # relative path to a parent directory
  - local: /opt/dbt/redshift # absolute path on the system
```

</File>

「ローカル」パッケージの使用を推奨する具体的なユースケースがいくつかあります。
1. **モノレポ** - モノレポ内のサブディレクトリにそれぞれネストされた複数のプロジェクトがある場合。「ローカル」パッケージを使用すると、プロジェクトを統合して、協調的な開発とデプロイメントを実現できます。
2. **変更のテスト** - あるプロジェクトまたはパッケージの変更を、それを使用する下流のプロジェクトまたはパッケージのコンテキスト内でテストします。インストールを一時的に「ローカル」パッケージに切り替えることで、前者に変更を加え、後者ですぐにテストして、反復処理を迅速化できます。これは、Python の[編集可能なインストール](https://pip.pypa.io/en/stable/topics/local-project-installs/)に似ています。
3. **ネストされたプロジェクト** - [`dbt-utils` パッケージ内の統合テスト](https://github.com/dbt-labs/dbt-utils/tree/main/integration_tests)のような、ユーティリティ マクロのプロジェクトのフィクスチャとテストを定義するネストされたプロジェクトがある場合。


## どのようなパッケージが利用可能ですか？
公開されているdbtパッケージのライブラリを確認するには、[dbt Hub](https://hub.getdbt.com)をご覧ください。

## 高度なパッケージ構成
### パッケージの更新
`packages.yml` ファイルのバージョンまたはリビジョンを更新しても、dbt プロジェクトでは自動的に更新されません。パッケージを更新するには、`dbt deps` を実行する必要があります。また、このパッケージ内のモデルの[完全更新](/reference/commands/run) も実行する必要があるかもしれません。

### パッケージのアンインストール
`packages.yml` ファイルからパッケージを削除しても、`dbt_packages/` ディレクトリに残っているため、dbt プロジェクトからは自動的に削除されません。パッケージを完全にアンインストールするには、次のいずれかの手順を実行してください。
* `dbt_packages/` 内のパッケージディレクトリを削除する。または
* `dbt clean` を実行してすべてのパッケージ（およびコンパイル済みのモデル）を削除し、その後 `dbt deps` を実行する。

### パッケージの固定

v1.7 以降では、[`dbt deps`](/reference/commands/deps) を実行すると、`p​​ackages.yml` が記録されている _project_root_ に `package-lock.yml` ファイルを作成または更新することで、各パッケージが「固定」されます。

- `package-lock.yml` ファイルには、インストールされているすべてのパッケージの記録が含まれます。
- 後続の `dbt deps` 実行で `dependencies.yml` または `packages.yml` に変更がない場合、dbt-core は `package-lock.yml` からインストールします。

たとえば、ブランチ名を使用した場合、`package-lock.yml` ファイルはヘッドコミットに固定されます。バージョン範囲を使用した場合、最新リリースに固定されます。どちらの場合も、後続のコミットまたはバージョンはインストールされません。新しいコミットまたはバージョンを取得するには、`dbt deps --upgrade` を実行するか、.gitignore ファイルに `package-lock.yml` を追加してください。

リビジョンを指定せずに `git` 構文を使用してパッケージをインストールすると、dbt から警告が表示されます (下記参照)。

### パッケージの設定
`dbt_project.yml` ファイルから、パッケージ内のモデルとシードを次のように設定できます。

<File name='dbt_project.yml'>

```yml

vars:
  snowplow:
    'snowplow:timezone': 'America/New_York'
    'snowplow:page_ping_frequency': 10
    'snowplow:events': "{{ ref('sp_base_events') }}"
    'snowplow:context:web_page': "{{ ref('sp_base_web_page_context') }}"
    'snowplow:context:performance_timing': false
    'snowplow:context:useragent': false
    'snowplow:pass_through_columns': []

models:
  snowplow:
    +schema: snowplow

seeds:
  snowplow:
    +schema: snowplow_seeds
```

</File>

たとえば、データセット固有のパッケージを使用する場合、生データを含むテーブルの名前の変数を設定する必要がある場合があります。

`dbt_project.yml` ファイルで行った設定は、パッケージ内の設定（パッケージの `dbt_project.yml` ファイル内、または設定ブロック内）をオーバーライドします。

### ピン留めされていない Git パッケージの指定

プロジェクトで「ピン留めされていない」 Git パッケージを指定した場合、次のような警告が表示されることがあります:

```
The git package "https://github.com/dbt-labs/dbt-utils.git" is not pinned.
This can introduce breaking changes into your project without warning!
```

この警告は、パッケージ仕様で `warn-unpinned: false` を設定することで非表示にすることができます。**注:** これは推奨されません。

<File name='packages.yml'>

```yaml
packages:
  - git: https://github.com/dbt-labs/dbt-utils.git
    warn-unpinned: false
```

</File>
