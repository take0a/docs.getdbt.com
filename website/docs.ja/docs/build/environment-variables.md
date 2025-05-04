---
title: 環境変数
id: "environment-variables"
description: "Use environment variables to customize the behavior of your dbt project."
---

環境変数を使用すると、プロジェクトの実行場所に応じて dbt プロジェクトの動作をカスタマイズできます。
プロジェクトコードで jinja 関数 `{{env_var('DBT_KEY','OPTIONAL_DEFAULT')}}` を呼び出す方法の詳細については、[env_var](/reference/dbt-jinja-functions/env_var) のドキュメントを参照してください。

:::info 環境変数の命名とプレフィックス

dbt Cloud の環境変数には、`DBT_`、`DBT_ENV_SECRET_`、または `DBT_ENV_CUSTOM_ENV_` のいずれかのプレフィックスを付ける必要があります。
環境変数のキーは大文字で、大文字と小文字が区別されます。
プロジェクトのコードで `{{env_var('DBT_KEY')}}` を参照する場合、キーは dbt Cloud の UI で定義された変数と完全に一致する必要があります。

:::

### 環境変数の設定と上書き

**優先順位**

環境変数の値は、dbt Cloud 内の複数の場所で設定できます。そのため、dbt Cloud は環境変数を以下の優先順位（低い順から高い順）に従って解釈します。

 <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/env-var-precdence.png" title="Environment variables order of precedence"/>

環境変数には4つのレベルがあります。

1. コード内の Jinja 関数 `env_var` に渡されるオプションのデフォルト引数。オーバーライド可能です。(_最低優先順位_)
2. プロジェクト全体のデフォルト値。オーバーライド可能です。
3. 環境レベル。さらにオーバーライド可能です。
4. ジョブレベル（ジョブオーバーライド）または個々の開発者の IDE 内（個人オーバーライド）。(_最高優先順位_)

**プロジェクトおよび環境レベルでの環境変数の設定**

プロジェクトおよび環境レベルで環境変数を設定するには、左上の **Deploy** をクリックし、**Environments** を選択します。**Environments Variables** をクリックして、環境変数を追加および更新します。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/navigate-to-env-vars.png" title="Environment variables tab"/>



`Project Default` 列があることに気づくでしょう。これは、コードの実行場所に関係なく、プロジェクト全体で保持される値を設定するのに最適な場所です。包括的なデフォルトを指定したり、プロジェクト全体のトークンやシークレットを追加したりする場合は、この値を設定することをお勧めします。

`Project Default` 列の右側には、すべての環境が表示されます。環境レベルで設定された値は、プロジェクトレベルのデフォルト値よりも優先されます。例えば、ステージング環境と本番環境で環境値を異なる方法で解釈するようにdbt Cloudに指示できます。


<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/project-environment-view.png" title="Setting project level and environment level values"/>


**ジョブレベルでの環境変数のオーバーライド**

同じ環境で複数のジョブを実行し、ジョブごとに環境変数の解釈を変えたい場合があります。

ジョブの設定または編集時に、環境レベルまたはプロジェクトレベルで定義された環境変数の値をオーバーライドできるセクションが表示されます。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/job-override.gif" title="Navigating to environment variables job override settings"/>


すべてのジョブは特定のデプロイメント環境で実行され、デフォルトでは、ジョブは実行環境の環境レベルで設定された値（または最も高い優先順位レベルに設定された値）を継承します。ジョブレベルで異なる値を設定したい場合は、値を編集してオーバーライドしてください。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/job-override.png" title="Setting a job override value"/>


**個人レベルでの環境変数のオーバーライド**

dbt 統合開発環境 (IDE) で開発する場合、環境変数の値を個人レベルでオーバーライドすることもできます。デフォルトでは、dbt Cloud はプロジェクトの開発環境で設定された環境変数の値を使用します。これらの値を確認およびオーバーライドするには、dbt Cloud で次の手順を実行します。
- 左側のメニューでアカウント名をクリックし、**Account settings** を選択します。
- **Your profile** セクションで **Credentials** をクリックし、プロジェクトを選択します。
- **Environment variables** セクションまでスクロールし、**Edit** をクリックして必要な変更を行います。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/personal-override.gif" title="Navigating to environment variables personal override settings"/>

オーバーライドを提供するために、開発者は編集して別の値を指定できます。これらの値は、IDEの「結果」タブと「コンパイル済みSQL」タブの両方で反映されます。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/personal-override.png" title="Setting a personal override value"/>

:::info 適切なカバレッジ
すべての環境変数にプロジェクトレベルのデフォルト値を設定していない場合、dbt Cloud があらゆるコンテキストで環境変数の値をどのように解釈すればよいか分からない可能性があります。このような場合、dbt は「環境変数が必要ですが、指定されていません」というコンパイルエラーをスローします。
:::

:::info IDE でセッション中に環境変数を変更する
IDE の使用中にセッション中に環境変数の値を変更した場合、変更を有効にするには IDE を更新する必要がある場合があります。
:::

開発中にIDEを更新するには、IDEの右下にある緑色の「準備完了」シグナルまたは赤色の「コンパイルエラー」メッセージのいずれかをクリックします。新しいモーダルがポップアップ表示されるので、「IDEを更新」ボタンを選択してください。これにより、環境変数の値が開発環境に読み込まれます。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/refresh-ide.png" title="Refreshing IDE mid-session"/>

プロジェクトの部分的な解析や、IDE でのセッション中の環境変数の変更に関して、既知の問題がいくつかあります。dbt プロジェクトが設定した値でコンパイルされない場合は、dbt プロジェクト内の `target/partial_parse.msgpack` ファイルを削除してみてください。これにより、dbt はプロジェクト全体を再コンパイルするようになります。

### シークレットの取り扱い

dbt Cloud では、すべての環境変数が保存時に暗号化されますが、dbt Cloud には、シークレット値やその他の機密情報を含む環境変数を管理するための追加機能があります。特定の環境変数をすべてのログとエラーメッセージから削除し、UI 上の値を難読化したい場合は、キーの先頭に「DBT_ENV_SECRET」を付けることができます。この機能は、「dbt v1.0」以降でサポートされています。


<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/DBT_ENV_SECRET.png" title="DBT_ENV_SECRET prefix obfuscation"/>

**注**: 環境変数は、[リポジトリのクローン作成用のGitトークン](/docs/build/environment-variables#clone-private-packages)を保存するために使用できます。セキュリティ対策を講じるため、Gitトークンの権限を読み取り専用に設定し、リポジトリへのアクセスが制限されたマシンアカウントまたはサービスユーザーのPATの使用を検討することをお勧めします。

### 特殊な環境変数

dbt Cloud には、いくつかの定義済み変数が組み込まれています。これらの変数は自動的に設定され、変更することはできません。

#### dbt Cloud IDE の詳細

dbt Cloud IDE では、以下の環境変数が自動的に設定されます。

- `DBT_CLOUD_GIT_BRANCH` - [dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) で開発用 Git ブランチ名を指定します。
- ブランチが変更されると、この変数も変更されます。
- ブランチ変更後に IDE を再起動する必要はありません。
- 現在、[dbt Cloud CLI](/docs/cloud/cloud-cli-installation) では使用できません。

ユースケース - これは、Git ブランチ名を [開発スキーマ](/docs/build/custom-schemas) のプレフィックスとして動的に使用する場合に役立ちます ( `{{ env_var ('DBT_CLOUD_GIT_BRANCH') }}` )。

#### dbt Cloud コンテキスト

以下の環境変数が自動的に設定されます。

- `DBT_ENV` - このキーは dbt Cloud アプリケーション用に予約されており、常に 'prod' に解決されます。デプロイメント実行専用です。
- `DBT_CLOUD_ENVIRONMENT_NAME` - `dbt` が実行されている dbt Cloud 環境の名前。
- `DBT_CLOUD_ENVIRONMENT_TYPE` - `dbt` が実行されている dbt Cloud 環境のタイプ。有効な値は `dev`、`staging`、または `prod` です。[一般的なデプロイメント環境](/docs/dbt-cloud-environments#types-of-environments) では値は空になりますので、`{{ env_var('DBT_CLOUD_ENVIRONMENT_TYPE', '') }}` のようなデフォルトを使用してください。
- `DBT_CLOUD_INVOCATION_CONTEXT` - `dbt` が呼び出されるコンテキストタイプ。値は `dev`、`staging`、`prod`、または `ci` です。

#### 実行の詳細

- `DBT_CLOUD_PROJECT_ID` - この実行の dbt Cloud プロジェクトの ID
- `DBT_CLOUD_JOB_ID` - この実行の dbt Cloud ジョブの ID
- `DBT_CLOUD_RUN_ID` - この特定の実行の ID
- `DBT_CLOUD_RUN_REASON_CATEGORY` - この実行のトリガーの「カテゴリ」（`scheduled`、`github_pull_request`、`gitlab_merge_request`、`azure_pull_request`、`other` のいずれか）
- `DBT_CLOUD_RUN_REASON` - この実行のトリガーの `カテゴリ` この実行の特定のトリガー（例: `スケジュール済み`、`メールアドレスによって開始`、または `API` 経由のカスタム）
- `DBT_CLOUD_ENVIRONMENT_ID` - この実行の環境の ID
- `DBT_CLOUD_ACCOUNT_ID` - この実行の dbt Cloud アカウントの ID

#### Git の詳細

_以下の変数は現在、Webhook 経由でトリガーされる GitHub、GitLab、Azure DevOps の PR ビルドでのみ利用可能です_

- `DBT_CLOUD_PR_ID` - 接続されたバージョン管理システムのプルリクエスト ID
- `DBT_CLOUD_GIT_SHA` - このプルリクエストビルドで実行されている Git コミット SHA


### 使用例

環境変数はさまざまな用途に使用でき、dbt Cloud で必要な操作をより簡単に実行するための強力さと柔軟性を提供します。

<Expandable alt_header="プライベートパッケージのクローン">

シークレットを環境変数として設定できるようになったため、パッケージのHTTPS URLにgitトークンを渡して、プライベートリポジトリをオンザフライでクローンできるようになりました。[プライベートパッケージのクローン](/docs/build/packages#private-packages)の有効化について詳しくは、こちらをご覧ください。

</Expandable>

<Expandable alt_header="Snowflake接続でウェアハウスを動的に設定する">

環境変数を使用すると、ジョブに応じて Snowflake 仮想ウェアハウスのサイズを動的に変更できます。プロジェクト接続でウェアハウス名を直接呼び出す代わりに、実行時に特定の仮想ウェアハウスに設定される環境変数を参照できます。

例えば、フルリフレッシュジョブを XL ウェアハウスで実行し、増分ジョブは中規模ウェアハウスでのみ実行する必要があるとします。両方のジョブは同じ dbt Cloud 環境に設定されています。接続構成では、環境変数を使用してウェアハウス名を `{{env_var('DBT_WAREHOUSE')}}` に設定できます。その後、ジョブ設定で、ジョブのワークロードに応じて `DBT_WAREHOUSE` 環境変数に異なる値を設定できます。

現在、1 回の実行で複数のモデルにわたって環境変数を動的に設定することはできません。これは、各 env_var が実行期間全体にわたって 1 つの設定値しか持てないためです。

**注** &mdash; この方法は、Databricks SQL Warehouse でも使用できます。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/Environment Variables/warehouse-override.png" title="Adding environment variables to your connection credentials"/>

:::info 環境変数とSnowflake OAuthの制限事項
環境変数は、ユーザー名/パスワードとキーペア（スケジュールされたジョブを含む）で正常に動作します。これは、dbt Coreが自動生成された`profiles.yml`に挿入されたJinjaを使用し、それを解決して`env_var`ルックアップを実行するためです。

ただし、Snowflake OAuth接続設定で環境変数を使用する場合、いくつかの制限があります。

- アカウント/ホストフィールドでは使用できませんが、データベース、ウェアハウス、およびロールでは使用できます。これらのフィールドでは、[拡張属性を使用](/docs/deploy/deploy-environments#deployment-connection)。

注意点として、アカウント/ホストフィールドに環境変数を指定すると、Snowflake OAuth接続は**接続に失敗します**。これは、フィールドがJinjaレンダリングを通過できないために発生します。dbt Cloudは、リテラルの`env_var`コードを`{{ env_var("DBT_ACCOUNT_HOST_NAME") }}.snowflakecomputing.com`のようなURL文字列にそのまま渡しますが、これは無効なホスト名です。代わりに[拡張属性](/docs/deploy/deploy-environments#deployment-credentials)を使用してください。
:::

</Expandable>

<Expandable alt_header="実行メタデータを監査する">

以下に、実行ごとに自動的に設定されるdbt Cloud実行IDを使用した、もう一つの効果的な例を示します。この追加データフィールドは、監査とデバッグに使用できます:

```sql

{{ config(materialized='incremental', unique_key='user_id') }}

with users_aggregated as (

    select
        user_id,
        min(event_time) as first_event_time,
        max(event_time) as last_event_time,
        count(*) as count_total_events

    from {{ ref('users') }}
    group by 1

)

select *,
    -- Inject the run id if present, otherwise use "manual"
    '{{ env_var("DBT_CLOUD_RUN_ID", "manual") }}' as _audit_run_id

from users_aggregated
```

</Expandable>

<Expandable alt_header="セマンティック レイヤーの資格情報を構成する">

import SLEnvVars from '/snippets/_sl-env-vars.md';

<SLEnvVars/>

</Expandable>
