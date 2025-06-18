---
title: "About the dbt Snowflake Native App"
id: "snowflake-native-app"
description: "An overview of the dbt Snowflake Native App for dbt accounts"
pagination_prev: null
pagination_next: "docs/cloud-integrations/set-up-snowflake-native-app"
---

# dbt Snowflakeネイティブアプリについて <Lifecycle status='preview' />

dbt Snowflakeネイティブアプリは、SnowflakeネイティブアプリフレームワークとSnowparkコンテナサービスを搭載しており、<Constant name="cloud" />エクスペリエンスをSnowflakeユーザーインターフェースに拡張します。Snowflakeログインで、以下の3つのエクスペリエンスにアクセスできます。

- **<Constant name="explorer" />** - [<Constant name="explorer" />](/docs/explore/explore-projects) の埋め込みバージョン
- **Ask dbt** - [<Constant name="semantic_layer" />](/docs/use-dbt-semantic-layer/dbt-sl)、OpenAI、Snowflake Cortexを搭載したdbt支援チャットボット
- **オーケストレーションの可観測性** - [ジョブ実行履歴](/docs/deploy/run-visibility)のビューと、[デプロイジョブ](/docs/deploy/deploy-jobs)をトリガーするSnowflakeタスクを作成するためのサンプルコード。

これらのエクスペリエンスにより、<Constant name="cloud" />で構築された機能を、従来dbtプロジェクトの下流で作業してきたBIアナリストや技術関係者などのユーザーに拡張できます。

インストール手順については、[dbt Snowflakeネイティブアプリのセットアップ](/docs/cloud-integrations/set-up-snowflake-native-app)を参照してください。

## アーキテクチャ

dbt Snowflakeネイティブアプリの動作に関連するツールは3つあります。

| Tool                               | Description |
|------------------------------------|-------------|
| 消費者の Snowflake アカウント | Snowpark Container Services を利用したネイティブアプリがインストールされている場所です。<br /><br />ネイティブアプリは、[Snowflake の外部ネットワーク アクセス](https://docs.snowflake.com/en/developer-guide/external-network-access/external-network-access-overview) を使用して、<Constant name="cloud" /> API と Datadog API (ログ記録用) を呼び出します。<br /><br />**Ask dbt** チャットボットを実行するために、<Constant name="semantic_layer" /> は Cortex LLM にアクセスしてクエリを実行し、プロンプトに基づいてテキストを生成します。これは、ユーザーが <Constant name="semantic_layer" /> 環境をセットアップするときに構成されます。|
| dbt 製品 Snowflake アカウント | ネイティブ アプリ アプリケーション パッケージがホストされ、コンシューマー アカウントに配布される場所です。<br /><br />コンシューマーのイベント テーブルは、アプリケーションの監視とログ記録のためにこのアカウントと共有されます。 |
| 消費者の <Constant name="cloud" /> アカウント | ネイティブ アプリは、メタデータの <Constant name="cloud" /> API と対話し、<Constant name="semantic_layer" /> クエリを処理して、ネイティブ アプリのエクスペリエンスを強化します。<br /> <br /> また、<Constant name="cloud" /> アカウントは、消費者の Snowflake アカウントを呼び出して、ウェアハウスを利用してオーケストレーション用の dbt クエリを実行し、Cortex LLM Arctic を使用して **Ask dbt** チャットボットを強化します。 |

The following diagram provides an illustration of the architecture:

<Lightbox src="/img/docs/cloud-integrations/architecture-dbt-snowflake-native-app.png" title="Architecture of dbt and Snowflake integration"/>


## アクセス

通常のSnowflakeログイン認証方法を使用して、dbt Snowflakeネイティブアプリにログインしてください。Snowflakeユーザーには、_[開発者ライセンス](/docs/cloud/manage-access/seats-and-users)_を持つ対応する<Constant name="cloud" />ユーザーがいる必要があります。以前の機能[プレビュー](/docs/dbt-versions/product-lifecycles#dbt-cloud)では、これは必須ではありませんでした。

Snowflakeネイティブアプリが既に構成されている場合は、アプリから<Constant name="cloud" />に次回アクセスする際に[認証情報のリンク](#link-credentials)を求めるメッセージが表示されます。これは1回限りのプロセスです。

## 調達
dbt Snowflakeネイティブアプリは、[Snowflakeマーケットプレイス](https://app.snowflake.com/marketplace/listing/GZTYZSRT2UA/dbt-labs-dbt)から入手できます。ご購入いただくと、ネイティブアプリへのアクセスと、Enterpriseプランの<Constant name="cloud" />アカウントが付与されます。既存の<Constant name="cloud" /> Enterpriseのお客様もご利用いただけます。ご興味をお持ちの場合は、Enterpriseアカウントマネージャーまでお問い合わせください。

ご興味をお持ちいただけましたら、詳細について[お問い合わせ](mailto:sales_snowflake_marketplace@dbtlabs.com)までご連絡ください。

## サポート
dbt Snowflakeネイティブアプリについてご質問がある場合は、[サポートチーム](mailto:dbt-snowflake-marketplace@dbtlabs.com)までお問い合わせください。ネイティブアプリのインストールに関する情報（<Constant name="cloud" /> アカウントIDとSnowflakeアカウントIDなど）をご提供ください。

## 制限事項
- ネイティブアプリは、[IP制限](/docs/cloud/secure/ip-restrictions)が有効になっている <Constant name="cloud" /> アカウントをサポートしません。

## 認証情報のリンク

ネイティブアプリにアクセスするすべてのSnowflakeユーザーは、[開発者ライセンスまたは読み取り専用ライセンス](/docs/cloud/manage-access/seats-and-users)を持つ<Constant name="cloud" />アカウントへのアクセス権も必要です。機能へのアクセスは、<Constant name="cloud" />ライセンスの種類によって異なります。

Snowflakeネイティブアプリが設定されている既存のアカウントの場合、ユーザーは次回ログイン時に<Constant name="cloud" />で認証するよう求められます。<Constant name="cloud" />にユーザーが登録されている場合、これは1回限りのプロセスです。<Constant name="cloud" />ユーザーが登録されていない場合はアクセスが拒否され、管理者が[ユーザーを作成](/docs/cloud/manage-access/invite-users)する必要があります。

1. Snowflakeネイティブアプリから<Constant name="cloud" />プラットフォームにアクセスしようとすると、アカウントのリンクを求めるメッセージが表示されます。

<Lightbox src="/img/docs/dbt-cloud/snowflake-link-account-prompt.png" width="90%" title="The Snowflake Native App prompt to link accounts" />

2. **アカウントをリンク** をクリックすると、<Constant name="cloud" /> の資格情報の入力を求められます。

<Lightbox src="/img/docs/dbt-cloud/snowflake-link-dbt-cloud.png" width="90%" title="The link accounts prompt" />
