---
title: "Set up the dbt Snowflake Native App"
description: "Learn how to set up the dbt Snowflake Native App"
pagination_prev: "docs/cloud-integrations/snowflake-native-app"
pagination_next: null
---

# dbt Snowflakeネイティブアプリをセットアップする <Lifecycle status='preview' />

[dbt Snowflake ネイティブ アプリ](/docs/cloud-integrations/snowflake-native-app) は、Snowflake ユーザー インターフェース内で、<Constant name="explorer" />、**Ask dbt** チャットボット、および <Constant name="cloud" /> のオーケストレーション オブザーバビリティ機能を有効にします。

この統合を設定するには、<Constant name="cloud" /> と Snowflake の両方を構成します。大まかな手順は次のとおりです。

1. **Ask dbt** 構成をセットアップします。
1. Snowflake を構成します。
1. <Constant name="cloud" /> を構成します。
1. dbt Snowflake ネイティブ アプリを購入してインストールします。
1. アプリを構成します。
1. アプリのインストールが成功したことを確認します。
1. 新しいユーザーをアプリにオンボーディングします。

ネイティブ アプリの公開リストを購入した場合、手順の順序が若干異なります。まずネイティブ アプリを購入し、前提条件を満たしてから、残りの手順を順番に完了します。

## 前提条件
<Constant name="cloud" /> と Snowflake の前提条件は次のとおりです。

### dbt

- AWS リージョンまたは Azure リージョン内のエンタープライズプランの <Constant name="cloud" /> アカウントが必要です。まだお持ちでない場合は、[お問い合わせ](mailto:sales_snowflake_marketplace@dbtlabs.com) から開始してください。
- 現在、Azure ST インスタンスでは <Constant name="semantic_layer" /> はご利用いただけません。また、このアカウントがないと、dbt Snowflake ネイティブアプリの **Ask dbt** チャットボットは機能しません。
- <Constant name="cloud" /> アカウントには、[サービストークン](/docs/dbt-cloud-apis/service-tokens) を作成する権限が必要です。詳細については、[エンタープライズ権限](/docs/cloud/manage-access/enterprise-permissions) をご覧ください。
- [<Constant name="semantic_layer" /> が設定](/docs/use-dbt-semantic-layer/setup-sl)され、メトリックが宣言された <Constant name="cloud" /> プロジェクトがあります。
- [本番環境のデプロイメント環境](/docs/deploy/deploy-environments#set-as-production-environment) をセットアップしました。
- デプロイメント環境で、`docs generate` ステップを含むジョブ実行が少なくとも 1 回成功しています。

### Snowflake

- Snowflake で **ACCOUNTADMIN** アクセス権が必要です。
- Snowflake アカウントには、ネイティブアプリ/SPCS 統合および NA/SPCS 構成へのアクセス権が必要です（パブリックプレビューは 6 月末に予定されています）。不明な場合は、Snowflake アカウントマネージャーにお問い合わせください。
- Snowflake アカウントは AWS リージョンに属している必要があります。Azure は現在、ネイティブアプリ/SPCS 統合ではサポートされていません。
- Snowflake の権限を通じて Snowflake Cortex にアクセスできる必要があります。[Snowflake Cortex はお客様のリージョンで利用可能です](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#availability)。これらがない場合、Ask dbt は動作しません。

## Ask dbt の設定

**Ask dbt** チャットボットを動かすために、<Constant name="cloud" /> と Snowflake Cortex を設定します。

1. <Constant name="cloud" /> で、<Constant name="semantic_layer" /> の設定を参照します。

    1. 左側のパネルに移動し、アカウント名をクリックします。そこから [アカウント設定] を選択します。
    1. 左側のサイドバーで [プロジェクト] を選択し、プロジェクトリストから dbt プロジェクトを選択します。

    1. [プロジェクトの詳細] パネルで、[GraphQL URL] オプションの下にある [<Constant name="semantic_layer" /> 設定の編集] リンクをクリックします。
1. **<Constant name="semantic_layer" /> 構成の詳細** パネルで、Snowflake の認証情報（Snowflake Cortex へのアクセスに使用）と、<Constant name="semantic_layer" /> を実行する環境を指定します。ユーザー名、ロール、環境を一時的な場所に保存し、後で使用できるようにします。

    <Lightbox src="/img/docs/cloud-integrations/semantic_layer_configuration.png" width="100%" title="Semantic Layer credentials"/>

1. Snowflakeで、SLとデプロイメントユーザーにSnowflake Cortexを使用する権限が付与されていることを確認してください。詳細については、Snowflakeドキュメントの[必要な権限](https://docs.snowflake.com/en/user-guide/snowflake-cortex/llm-functions#required-privileges)を参照してください。

    デフォルトでは、すべてのユーザーがSnowflake Cortexにアクセスできます。これが無効になっている場合は、Snowflake SQLワークシートを開き、以下のステートメントを実行してください。

    ```sql
    create role cortex_user_role;
    grant database role SNOWFLAKE.CORTEX_USER to role cortex_user_role;
    grant role cortex_user_role to user SL_USER;
    grant role cortex_user_role to user DEPLOYMENT_USER;
    ```

    `SNOWFLAKE.CORTEX_USER`、`DEPLOYMENT_USER`、および `SL_USER` を、ご使用の環境に適した文字列に置き換えてください。

## dbt を構成する
アプリケーションをセットアップするには、<Constant name="cloud" /> から以下の情報を収集します。

1. 左側のパネルに移動し、アカウント名をクリックします。そこから **アカウント設定** を選択します。次に、**API トークン > サービス トークン** をクリックします。dbt Snowflake ネイティブアプリでアクセスするすべてのプロジェクトへのアクセス権を持つサービストークンを作成します。以下の権限セットを付与します。
    - **マーケットプレイスアプリの管理**
    - **ジョブ管理者**
    - **メタデータのみ**
    - **<Constant name="semantic_layer" /> のみ**

    トークン情報は、後でネイティブアプリの構成時に使用するため、一時的な場所に保存してください。

    以下は、すべてのプロジェクトに権限セットを付与する例です。

    <Lightbox src="/img/docs/cloud-integrations/example-snowflake-native-app-service-token.png" title="Example of a new service token for the dbt Snowflake Native App"/>

1. 左側のサイドバーから [**アカウント**] を選択し、ネイティブアプリの設定時に後で使用できるように、以下の情報を一時的な場所に保存します。
    - **アカウントID** - <Constant name="cloud" /> アカウントを表す数値文字列。
    - **アクセス URL** - 北米のマルチテナントアカウントをお持ちの場合は、アクセス URL として `cloud.getdbt.com` を使用してください。その他のリージョンについては、[アクセス、リージョン、および IP アドレス](/docs/cloud/about-cloud/access-regions-ip-addresses) を参照し、表から必要なアクセス URL を調べてください。

## dbt Snowflakeネイティブアプリをインストールします
1. dbt Snowflakeネイティブアプリのリストを参照します。
    - **プライベートリスト** (推奨) - 送信されたメールのリンクを使用します。
    - **パブリックリスト** - [Snowflake Marketplace](https://app.snowflake.com/marketplace/listing/GZTYZSRT2R3) に移動します。
1. リストの「**入手**」をクリックして、dbt Snowflakeネイティブアプリをインストールします。インストールには数分かかる場合があります。インストールが完了すると、メールが送信されます。

アプリケーションを変更して、インストールのためにウェアハウスへのアクセスを許可するかどうかを確認するメッセージが表示されます。dbt Labsは、必要がない限りアプリケーション名を変更しないことを強くお勧めします。
1. dbt Snowflakeネイティブアプリが正常にインストールされたら、モーダルウィンドウで「**構成**」をクリックします。

## dbt Snowflakeネイティブアプリの構成

1. **「dbtのアクティベート」** ページで、**ステップ1：アカウント権限の付与** の **「付与」** をクリックします。
1. 権限が正常に付与されたら、**ステップ2：接続の許可** の **「確認」** をクリックします。

    **「<Constant name="cloud" /> 外部アクセス統合への接続」** の手順を実行します。事前に収集した <Constant name="cloud" /> アカウント情報が必要になります。プロンプトが表示されたら、アカウントID、アクセスURL、APIサービストークンを **シークレット値** として入力します。
1. **「dbtのアクティベート」** ページで、<Constant name="cloud" /> 外部アクセス統合への接続が確立されたら、**「アクティベート」** をクリックします。必要なSnowflakeサービスとコンピューティングリソースが起動するまで数分かかる場合があります。
1. アクティベーションが完了したら、**「テレメトリ」** タブを選択し、`INFO` ログを共有するオプションを有効にします。このオプションが表示されるまで時間がかかる場合があります。これは、Snowflakeがイベントテーブルを作成して共有できるようにするためです。
1. オプションが有効になったら、「**アプリを起動**」をクリックします。次に、Snowflakeの認証情報を使用してアプリにログインします。

    ログインページではなく、Snowsightワークシートにリダイレクトされる場合は、アプリのインストールが完了していないことを意味します。通常、ページを更新することでこの問題を解決できます。

    以下は、構成後のdbt Snowflakeネイティブアプリの例です。

    <Lightbox src="/img/docs/cloud-integrations/example-dbt-snowflake-native-app.png" title="Example of the dbt Snowflake Native App"/>

## アプリが正常にインストールされたことを確認する

アプリが正常にインストールされたことを確認するには、サイドバーから次のいずれかを選択します。

- **Explore** &mdash; <Constant name="explorer" /> を起動し、dbt プロジェクト情報にアクセスできることを確認します。
- **Jobs** &mdash; dbt ジョブの実行履歴を確認します。
- **Ask dbt** &mdash; 提案されたプロンプトのいずれかをクリックして、チャットボットに質問します。dbt プロジェクトに定義されている指標の数によっては、dbt が Retrieval Augmented Generation (RAG) を構築しているため、初回の **Ask dbt** の読み込みに数分かかる場合があります。次回の起動では読み込みが速くなります。

以下は、上部近くに提案されたプロンプトが表示されている **Ask dbt** チャットボットの例です。 

<Lightbox src="/img/docs/cloud-integrations/example-ask-dbt-native-app.png" title="Example of the Ask dbt chatbot"/>


## 新規ユーザーのオンボーディング
1. Snowflake のサイドバーから、**[データ製品] > [アプリ]** を選択します。リストから **dbt** を選択して、アプリの構成ページを開きます。次に、右上の [アクセスの管理]** をクリックして、新規ユーザーをアプリケーションにオンボーディングします。アプリケーションへのアクセスは許可するが、構成の編集権限は付与しない適切なロールに **APP_USER** ロールを付与します。構成の編集または削除権限を付与するロールには **APP_ADMIN** ロールを付与します。

1. 新規ユーザーは、共有された Snowflake アプリの URL を使用するか、アプリの構成ページで **[アプリの起動]** をクリックすることで、アプリにアクセスできます。


## FAQs

<Expandable alt_header="Snowflake Marketplaceからdbt Snowflake Nativeアプリをインストールできない" >

<Constant name="cloud" /> Snowflake ネイティブ アプリは、Snowflake 無料トライアル アカウントでは利用できません。

</Expandable>

<Expandable alt_header="Ask dbtから `Unable to access schema dbt_sl_llm` というエラーメッセージを受け取りました" >

SL ユーザーに `dbt_sl_llm` スキーマへのアクセスが許可されていることを確認し、スキーマからの読み取りと書き込みに必要なすべての権限があることを確認します。

</Expandable>

<Expandable alt_header="ネイティブアプリで使用されるdbt構成オプションを更新する必要がある" >

<Constant name="cloud" /> アカウントID、アクセスURL、またはAPIサービストークンが更新された場合は、dbt Snowflakeネイティブアプリの構成を更新する必要があります。Snowflakeでアプリの構成ページに移動し、既存の構成を削除してください。新しい構成を追加し、Snowsightのアプリケーションデータベースで `CALL app_public.restart_app();` を実行してください。
</Expandable>

<Expandable alt_header="ネイティブ アプリでは環境変数はサポートされていますか?" >

[環境変数](/docs/build/environment-variables)（`{{env_var('DBT_WAREHOUSE') }}` など）は、<Constant name="semantic_layer" /> ではまだサポートされていません。「Ask dbt」機能を使用するには、代わりに実際の認証情報を使用する必要があります。
</Expandable>
