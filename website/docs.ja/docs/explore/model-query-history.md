---
title: "Model query history"
sidebar_label: "Model query history"
description: "Import and auto-generate exposures from dashboards and understand how models are used in downstream tools for a richer lineage."
image: /img/docs/collaborate/dbt-explorer/model-query-queried-models.jpg
---

# モデルクエリ履歴 <Lifecycle status="managed,managed_plus" />

<IntroText>
モデル クエリ履歴は、データ チームがクエリ ログを分析してモデルの使用状況を追跡するのに役立ちます。
</IntroText>

モデルクエリ履歴を使用すると、次のことが可能になります。

- データウェアハウスのクエリログに基づいて、モデルの使用クエリ数を表示します。
- データチームにインサイトを提供し、時間とインフラストラクチャの支出を、価値の高いデータ製品に集中させることができます。
- アナリストが、他のユーザーが最もよく使用しているモデルを見つけられるようにします。

モデルクエリ履歴は、データウェアハウス内のクエリログテーブルに対する単一の使用クエリを毎日集計することで作成されます。

<Expandable alt_header="消費クエリとは何ですか?">

消費クエリとは、特定の期間に dbt プロジェクト内でモデルを使用したクエリの指標です。モデルの消費量を測定するために `select` ステートメントのみをフィルタリングし、dbt モデルのビルドとテストの実行は除外します。

たとえば、`model_super_santi` が過去 1 週間に 10 回クエリされた場合、その期間の消費クエリは 10 件とカウントされます。
</Expandable>

:::info Snowflake（Enterprise 層以上）と BigQuery のサポート

Snowflake ユーザー向けのモデルクエリ履歴は、**Enterprise 層以上でのみ利用可能**です。この機能は BigQuery もサポートしています。その他のプラットフォームも近日中にサポート予定です。
:::

## 前提条件

これらの機能にアクセスするには、以下の要件を満たしている必要があります。

1. [エンタープライズプラン](https://www.getdbt.com/pricing/)の<Constant name="cloud" />アカウントをお持ちであること。シングルテナントアカウントの場合は、設定についてアカウント担当者にお問い合わせください。
2. 調査するプロジェクトごとに[本番環境](https://docs.getdbt.com/docs/deploy/deploy-environments#set-as-production-environment)デプロイメント環境をセットアップし、少なくとも1つのジョブ実行が成功していること。
3. プロジェクト設定または本番環境設定を編集するための、<Constant name="cloud" />の[管理者権限](/docs/cloud/manage-access/enterprise-permissions)を持っていること。
4. データウェアハウスとしてSnowflakeまたはBigQueryを使用し、[クエリ履歴権限](#snowflake-model-query-history)を有効にするか、管理者に依頼して有効化できます。その他のデータプラットフォームのサポートも近日中に開始予定です。
   - Snowflakeユーザーの場合：Snowflake Enterprise以上のサブスクリプションが必要です。

## dbt でクエリ履歴を有効にする

<Constant name="cloud" /> でモデルクエリ履歴を有効にするには、次の手順に従います。

1. **デプロイ** に移動し、**環境** を選択します。
2. **PROD** とマークされた環境を選択し、**設定** をクリックします。
3. **編集** をクリックし、**クエリ履歴** セクションまでスクロールして、クエリ履歴の切り替えを有効にします。緑色で右側に表示されている場合は、有効です。
4. **権限のテスト** ボタンをクリックして、デプロイメント資格情報の権限がクエリ履歴をサポートするのに十分であることを確認します。
5. <Constant name="cloud" /> は、新しい環境ではクエリ履歴を自動的に有効にします。クエリ履歴によるデータの取得に失敗した場合、意図しないウェアハウスコストの発生を防ぐために、<Constant name="cloud" /> はクエリ履歴を自動的に無効にします。
   - 失敗が一時的な場合（ネットワークタイムアウトなど）、<Constant name="cloud" /> は再試行することがあります。
   - 問題が永続的な場合（権限不足など）、<Constant name="cloud" /> はクエリ履歴を直ちに無効にします。

   再度有効にするには、[dbt サポート](mailto:support@getdbt.com) までお問い合わせください。

<DocCarousel slidesPerView={1}>

<Lightbox src="/img/docs/collaborate/dbt-explorer/enable-query-history.jpg" width="95%" title="Enable query history in your environment settings." />
<Lightbox src="/img/docs/collaborate/dbt-explorer/enable-query-history-success.jpg" width="95%" title="Example of permissions verified result after clicking Test Permissions." />

</DocCarousel>

## 認証情報による権限

このセクションでは、<Constant name="explorer" /> でモデルクエリ履歴を有効にして表示するために必要な権限と手順について説明します。

モデルクエリ履歴機能は、本番環境の認証情報を使用して、データウェアハウスのクエリログからメタデータを収集します。そのため、ウェアハウスに対する昇格された権限が必要になる場合があります。データプラットフォームの権限を変更する前に、<Constant name="cloud" /> で構成されている権限を確認してください。

1. 「**デプロイ**」に移動し、「**環境**」に移動します。
2. 「**PROD**」とマークされた環境を選択し、「**設定**」をクリックします。
3. 「**デプロイ認証情報**」の情報を確認します。
   - 注: クエリ履歴のクエリには、ウェアハウスのコストとクレジットの使用が発生します。
<Lightbox src="/img/docs/collaborate/dbt-explorer/model-query-credentials.jpg" width="50%" title="Confirm your deployment credentials in your environment settings page." />

4. これらの資格情報の権限をウェアハウスの権限とコピーまたは相互参照し、ユーザーに適切な権限を付与します。

#### Snowflake モデルのクエリ履歴
モデルクエリ履歴は、[Snowflake Enterprise レベル](https://docs.snowflake.com/en/user-guide/intro-editions#enterprise-edition) 以上のアカウントで利用可能なメタデータテーブル、`QUERY_HISTORY`、および `ACCESS_HISTORY` を使用します。本番環境の Snowflake ユーザーがデータを表示するには、`GOVERNANCE_VIEWER` 権限が必要です。
モデルクエリ履歴を有効にする前に、`ACCOUNTADMIN` が Snowflake で次の GRANT ステートメントを実行してアクセスを許可する必要があります。
     ```sql
     GRANT DATABASE ROLE SNOWFLAKE.GOVERNANCE_VIEWER TO ROLE <YOUR_DBT_CLOUD_DEPLOYMENT_ROLE>;
     ```
この権限がないと、モデルのクエリ履歴にデータが表示されません。詳細については、Snowflakeのドキュメント（[こちら](https://docs.snowflake.com/en/sql-reference/account-usage#enabling-other-roles-to-use-schemas-in-the-snowflake-database)）をご覧ください。. 

##### BigQuery モデルクエリ履歴
モデルクエリ履歴は、BigQuery の `INFORMATION_SCHEMA.JOBS` ビューのメタデータを使用します。このメタデータにアクセスするには、本番環境用に構成されたユーザーに、BigQuery プロジェクトに対する次の [IAM ロール](https://cloud.google.com/bigquery/docs/access-control) が付与されている必要があります。

       - `roles/bigquery.resourceViewer`
       - `roles/bigquery.jobs.create`

## エクスプローラーでクエリ履歴を表示

より効果的に探索を行うために、<Constant name="explorer" /> 内のさまざまな場所でモデルのクエリ履歴を表示できます。
- [パフォーマンスチャートから表示](#view-from-performance-charts)
* [プロジェクト系統から表示](#view-from-project-lineage)
- [モデルリストから表示](#view-from-model-list)

### パフォーマンスチャートからの表示

1. ナビゲーションの「**Explore**」リンクをクリックして、<Constant name="explorer" /> に移動します。
2. メインの「**Overview**」ページで、「**Project details**」セクションの「**Performance**」をクリックします。下にスクロールして、「**Most consumption models**」を表示します。
3. 右側のドロップダウンメニューを使用して、過去3か月までの期間を選択します。

<Lightbox src="/img/docs/collaborate/dbt-explorer/most-consumed-models.jpg" width="85%" title="View most consumed models on the 'Performance' page in dbt Explorer." />

4. 詳細を表示するにはモデルをクリックし、「**パフォーマンス**」タブに移動します。
5. 「**パフォーマンス**」タブで、「**モデルパフォーマンス**」セクションまで下にスクロールします。
6. 「**消費クエリ**」タブを選択すると、そのモデルの一定期間における消費クエリが表示されます。
<Lightbox src="/img/docs/collaborate/model-consumption-queries.jpg" width="90%" title="View consumption queries over time for a given model." />

### プロジェクト系統からの表示

1. プロジェクト系統でモデルを表示するには、メインの**概要ページ**に移動し、**プロジェクト系統**をクリックします。
2. 系統の左下にある**レンズ**をクリックし、**消費クエリ**を選択します。
<Lightbox src="/img/docs/collaborate/dbt-explorer/model-consumption-lenses.jpg" width="85%" title="View model consumption queries in your lineage using the 'Lenses' feature." />

3. 系統図の各モデルの上に、消費クエリ数を示す小さな赤いボックスが表示されます。各モデルの数字は、過去30日間のモデル消費量を表しています。

### モデルリストからの表示

1. モデルのリストを表示するには、メインの**概要ページ**に移動します。
2. 左側のナビゲーションで**リソース**タブに移動し、**モデル**をクリックしてモデルのリストを表示します。
3. モデルの消費クエリ数を確認し、消費量の多い順または少ない順に並べ替えることができます。各モデルの消費クエリ数は、過去30日間の消費量を表します。
<Lightbox src="/img/docs/collaborate/dbt-explorer/model-consumption-list.jpg" width="85%" title="View models consumption in the 'Models' list page under the 'Consumption' column." />
