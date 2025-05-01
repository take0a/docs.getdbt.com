---
title: "dbt Cloud セマンティック レイヤーと Snowflake のクイックスタート"
id: sl-snowflake-qs
description: "このガイドを使用して、指標を構築および定義し、dbt Cloud Semantic Layer を設定し、Google スプレッドシートを使用してクエリを実行します。"
sidebar_label: "Quickstart with the dbt Semantic Layer and Snowflake"
meta:
  api_name: dbt Semantic Layer APIs
icon: 'guides'
hide_table_of_contents: true
tags: ['Semantic Layer', 'Snowflake', 'dbt Cloud','Quickstart']
keywords: ['dbt Semantic Layer','Metrics','dbt Cloud', 'Snowflake', 'Google Sheets']
level: 'Intermediate'
recently_updated: true
---

<!-- The below snippets (or reusables) can be found in the following file locations in the docs code repository) -->
import CreateModel from '/snippets/_sl-create-semanticmodel.md';
import DefineMetrics from '/snippets/_sl-define-metrics.md';
import ConfigMetric from '/snippets/_sl-configure-metricflow.md';
import TestQuery from '/snippets/_sl-test-and-query-metrics.md';
import ConnectQueryAPI from '/snippets/_sl-connect-and-query-api.md';
import RunProdJob from '/snippets/_sl-run-prod-job.md';
import SlSetUp from '/snippets/_new-sl-setup.md'; 

## はじめに

[MetricFlow](/docs/build/about-metricflow) を基盤とする [dbt セマンティックレイヤー](/docs/use-dbt-semantic-layer/dbt-sl) は、主要なビジネスメトリクスの設定を簡素化します。
定義を一元化し、コードの重複を回避し、下流ツールからメトリクスに簡単にアクセスできるようにします。
MetricFlow は、dbt プロジェクトでメトリクスを定義し、[MetricFlow コマンド](/docs/build/metricflow-commands) を使用して dbt Cloud でクエリを実行できるため、企業のメトリクス管理を容易にします。

import SLCourses from '/snippets/_sl-course.md';

<SLCourses/>

このクイックスタートガイドは、Snowflakeをデータプラットフォームとして使用しているdbt Cloudユーザー向けに設計されています。
指標の構築と定義、dbt Cloudプロジェクトでのdbtセマンティックレイヤーの設定、Googleスプレッドシートでの指標のクエリに焦点を当てています。

異なるデータプラットフォームを使用している場合も、このガイドに従うことができますが、特定のプラットフォームに合わせて設定を変更する必要があります。
詳細については、[異なるプラットフォームのユーザー](#for-users-on-different-data-platforms)セクションをご覧ください。

### 前提条件

- すべてのデプロイメントには、[dbt Cloud](https://www.getdbt.com/signup/) トライアル、チーム、またはエンタープライズアカウントが必要です。
- プランに応じた適切な [dbt Cloud ライセンス](/docs/cloud/manage-access/seats-and-users) と [権限](/docs/cloud/manage-access/enterprise-permissions) が必要です。

  <DetailsToggle alt_header="ライセンスと権限に関する詳細情報">  
  
  - エンタープライズ - アカウント管理者権限を持つ開発者ライセンス。
  または、開発者ライセンスを持ち、プロジェクト作成者、データベース管理者、または管理者権限が割り当てられた「オーナー」。
  - チーム - 開発者ライセンスを持つ「オーナー」アクセス。
  - トライアル - チームプランのトライアルでは自動的に「オーナー」アクセスが付与されます。
  
  </DetailsToggle>

- [Snowflakeのトライアルアカウント](https://signup.snowflake.com/)を作成します。
  - ACCOUNTADMINアクセス権を持つEnterprise Snowflakeエディションを選択します。
  クラウドプロバイダーを選択する際には、組織に関する問題を考慮し、Snowflakeの[クラウドプラットフォームの概要](https://docs.snowflake.com/en/user-guide/intro-cloud-platforms)を参照してください。
  - クラウドプロバイダーとリージョンを選択します。
  すべてのクラウドプロバイダーとリージョンが利用可能ですので、お好みのものを選択してください。
- SQLとdbtの基本的な知識。
例えば、以前にdbtを使用したことがある、または[dbt Fundamentals](https://learn.getdbt.com/courses/dbt-fundamentals)コースを修了しているなどです。


### 異なるデータプラットフォームをご利用のお客様へ

Snowflake以外のデータプラットフォームをご利用の場合も、このガイドは適用されます。
各プラットフォームの以下のタブに記載されているアカウント設定とデータロードの手順に従って、お使いのプラットフォームに合わせて設定を調整してください。

このガイドの残りの部分は、サポートされているすべてのプラットフォームに共通に適用され、dbtセマンティックレイヤーを最大限に活用できるようにします。

<Tabs>

<TabItem value="bq" label="BigQuery">

新しいタブを開き、アカウントの設定とデータの読み込み手順については、以下の簡単な手順に従ってください:

- [ステップ 2: 新しい GCP プロジェクトを作成する](https://docs.getdbt.com/guides/bigquery?step=2)
- [ステップ 3: BigQuery データセットを作成する](https://docs.getdbt.com/guides/bigquery?step=3)
- [ステップ 4: BigQuery 認証情報を生成する](https://docs.getdbt.com/guides/bigquery?step=4)
- [ステップ 5: dbt Cloud を BigQuery に接続する](https://docs.getdbt.com/guides/bigquery?step=5)

</TabItem>

<TabItem value="databricks" label="Databricks">

新しいタブを開き、アカウントの設定とデータの読み込み手順については、以下の簡単な手順に従ってください:

- [ステップ 2: Databricks ワークスペースを作成する](https://docs.getdbt.com/guides/databricks?step=2)
- [ステップ 3: データを読み込む](https://docs.getdbt.com/guides/databricks?step=3)
- [ステップ 4: dbt Cloud を Databricks に接続する](https://docs.getdbt.com/guides/databricks?step=4)

</TabItem>

<TabItem value="msfabric" label="Microsoft Fabric">

新しいタブを開き、以下の簡単な手順に従ってアカウントの設定とデータの読み込みを行ってください:

- [ステップ 2: Microsoft Fabric ウェアハウスにデータを読み込む](https://docs.getdbt.com/guides/microsoft-fabric?step=2)
- [ステップ 3: dbt Cloud を Microsoft Fabric に接続する](https://docs.getdbt.com/guides/microsoft-fabric?step=3)

</TabItem>

<TabItem value="redshift" label="Redshift">

新しいタブを開き、アカウントの設定とデータの読み込み手順については、以下の簡単な手順に従ってください:

- [ステップ 2: Redshift クラスターを作成する](https://docs.getdbt.com/guides/redshift?step=2)
- [ステップ 3: データを読み込む](https://docs.getdbt.com/guides/redshift?step=3)
- [ステップ 4: dbt Cloud を Redshift に接続する](https://docs.getdbt.com/guides/redshift?step=3)

</TabItem>

<TabItem value="starburst" label="Starburst Galaxy">

新しいタブを開き、アカウントの設定とデータのロード手順については、以下の簡単な手順に従ってください:

- [ステップ 2: Amazon S3 バケットにデータをロードする](https://docs.getdbt.com/guides/starburst-galaxy?step=2)
- [ステップ 3: Starburst Galaxy を Amazon S3 バケットのデータに接続する](https://docs.getdbt.com/guides/starburst-galaxy?step=3)
- [ステップ 4: Starburst Galaxy を使用してテーブルを作成する](https://docs.getdbt.com/guides/starburst-galaxy?step=4)
- [ステップ 5: dbt Cloud を Starburst Galaxy に接続する](https://docs.getdbt.com/guides/starburst-galaxy?step=5)

</TabItem>

</Tabs>

## 新しいSnowflakeワークシートを作成し、環境を設定します。

1. [トライアル版Snowflakeアカウント](https://signup.snowflake.com)にログインします。
2. Snowflakeユーザーインターフェース（UI）で、右上にある**+ワークシート**をクリックします。
3. **SQLワークシート** を選択して、新しいワークシートを作成します。

### Snowflake 環境の設定

ここで使用するデータは、CSV ファイルとしてパブリック S3 バケットに保存されています。以下の手順に従って、Snowflake アカウントでそのデータを準備し、アップロードしてください。

新しい仮想ウェアハウス、2 つの新しいデータベース（1 つは生データ用、もう 1 つは将来の dbt 開発用）、2 つの新しいスキーマ（1 つは `jaffle_shop` データ用、もう 1 つは `stripe` データ用）を作成します。

1. 新しい Snowflake SQL ワークシートのエディターに次の SQL コマンドを 1 つずつ入力して実行し、環境を設定します。

2. 各コマンドの UI 右上隅にある [実行] をクリックします:

```sql
-- Create a virtual warehouse named 'transforming'
create warehouse transforming;

-- Create two databases: one for raw data and another for analytics
create database raw;
create database analytics;

-- Within the 'raw' database, create two schemas: 'jaffle_shop' and 'stripe'
create schema raw.jaffle_shop;
create schema raw.stripe;
```

### Snowflakeへのデータのロード

環境設定が完了したら、データのロードを開始できます。
「jaffle_shop」スキーマとストライプスキーマを使用して、生のデータベース内で作業を行います。

1. 顧客テーブルを作成します。まず、Snowflakeワークシートのエディターですべてのコンテンツを削除（空の状態）します。
次に、次のSQLコマンドを実行して、「jaffle_shop」スキーマに顧客テーブルを作成します:

  ```sql
  create table raw.jaffle_shop.customers
  ( id integer,
    first_name varchar,
    last_name varchar
  );
  ```

  「テーブル `CUSTOMERS` が正常に作成されました。」というメッセージが表示されます。

2. データをロードします。テーブルを作成したら、エディタ内のすべてのコンテンツを削除します。
以下のコマンドを実行して、S3バケットから顧客テーブルにデータをロードします:

  ```sql
  copy into raw.jaffle_shop.customers (id, first_name, last_name)
  from 's3://dbt-tutorial-public/jaffle_shop_customers.csv'
  file_format = (
      type = 'CSV'
      field_delimiter = ','
      skip_header = 1
      );
  ```

  コマンドを実行すると確認メッセージが表示されます。

3. `orders` テーブルを作成します。エディタ内の内容をすべて削除します。以下のコマンドを実行して作成します。

  ```sql
  create table raw.jaffle_shop.orders
  ( id integer,
    user_id integer,
    order_date date,
    status varchar,
    _etl_loaded_at timestamp default current_timestamp
  );
  ```

  コマンドを実行すると確認メッセージが表示されます。

4. データをロードします。エディター内のすべてのコンテンツを削除し、次のコマンドを実行してデータをordersテーブルにロードします。

  ```sql
  copy into raw.jaffle_shop.orders (id, user_id, order_date, status)
  from 's3://dbt-tutorial-public/jaffle_shop_orders.csv'
  file_format = (
      type = 'CSV'
      field_delimiter = ','
      skip_header = 1
      );
  ```

  コマンドを実行すると確認メッセージが表示されます。

5. `payment`テーブルを作成します。エディタ内の内容をすべて削除します。以下のコマンドを実行してpaymentテーブルを作成します。

  ```sql
  create table raw.stripe.payment
  ( id integer,
    orderid integer,
    paymentmethod varchar,
    status varchar,
    amount integer,
    created date,
    _batched_at timestamp default current_timestamp
  );
  ```

  コマンドを実行すると確認メッセージが表示されます。

6. データをロードします。エディター内のすべてのコンテンツを削除します。以下のコマンドを実行して、支払いテーブルにデータをロードします。

  ```sql
  copy into raw.stripe.payment (id, orderid, paymentmethod, status, amount, created)
  from 's3://dbt-tutorial-public/stripe_payments.csv'
  file_format = (
      type = 'CSV'
      field_delimiter = ','
      skip_header = 1
      );
  ```

  コマンドを実行すると確認メッセージが表示されます。

7. データを検証します。これらのSQLクエリを実行して、データがロードされていることを確認します。それぞれの出力が以下の確認画像のように表示されることを確認してください。

  ```sql
  select * from raw.jaffle_shop.customers;
  select * from raw.jaffle_shop.orders;
  select * from raw.stripe.payment;
  ```

  <Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-snowflake-confirm.jpg" width="90%" title="The image displays Snowflake's confirmation output when data loaded correctly in the Editor." />

## dbt Cloud を Snowflake に接続する

dbt Cloud を Snowflake に接続するには 2 つの方法があります。
1 つ目は Partner Connect です。これは、新しい Snowflake トライアルアカウントから dbt Cloud アカウントを簡単に作成できる方法です。
2 つ目は、dbt Cloud アカウントを別途作成し、Snowflake への接続を自分で構築する方法です（手動で接続）。
すぐに使い始めたい場合は、dbt Labs では Partner Connect のご利用を推奨しています。
最初から設定をカスタマイズし、dbt Cloud のセットアップフローに慣れたい場合は、手動で接続することを推奨しています。

<Tabs>
<TabItem value="partner-connect" label="Use Partner Connect" default>

Partner Connect を使用すると、[Snowflake 接続](/docs/cloud/connect-data-platform/connect-snowflake)、[マネージドリポジトリ](/docs/cloud/git/managed-repository)、[環境](/docs/build/custom-schemas#managing-environments)、および認証情報を使用して、完全な dbt アカウントを作成できます。

1. Snowflake UI で、左上隅のホームアイコンをクリックします。左側のサイドバーで [**Data Products**] を選択します。次に、[**Partner Connect**] を選択します。スクロールするか、検索バーで「dbt」を検索して、dbt タイルを見つけます。タイルをクリックして dbt に接続します。

    <Lightbox src="/img/snowflake_tutorial/snowflake_partner_connect_box.png" title="Snowflake Partner Connect Box" />

    Snowflake UIのクラシックバージョンをご利用の場合は、アカウント上部のバーにある**Partner Connect**ボタンをクリックしてください。そこからdbtタイルをクリックすると、接続ボックスが開きます。

    <Lightbox src="/img/snowflake_tutorial/snowflake_classic_ui_partner_connect.png" title="Snowflake Classic UI - Partner Connect" />

2. 「**dbt に接続**」ポップアップで「**オプションの付与**」オプションを見つけ、「**RAW**」と「**ANALYTICS**」データベースを選択します。これにより、新しい dbt ユーザーロールに、選択した各データベースへのアクセスが許可されます。「**接続**」をクリックします。

    <Lightbox src="/img/snowflake_tutorial/snowflake_classic_ui_connection_box.png" title="Snowflake Classic UI - Connection Box" />

    <Lightbox src="/img/snowflake_tutorial/snowflake_new_ui_connection_box.png" title="Snowflake New UI - Connection Box" />

3. ポップアップが表示されたら、[**アクティブ化**] をクリックします:

<Lightbox src="/img/snowflake_tutorial/snowflake_classic_ui_activation_window.png" title="Snowflake Classic UI - Actviation Window" />

<Lightbox src="/img/snowflake_tutorial/snowflake_new_ui_activation_window.png" title="Snowflake New UI - Activation Window" />

4. 新しいタブが読み込まれると、フォームが表示されます。dbt Cloudアカウントを既に作成済みの場合は、アカウント名の入力を求められます。アカウントを作成していない場合は、アカウント名とパスワードの入力を求められます。

<Lightbox src="/img/snowflake_tutorial/dbt_cloud_account_info.png" title="dbt Cloud - Account Info" />

5. フォームに記入し、「**登録完了**」をクリックすると、dbt Cloud に自動的にログインします。

6. 左側のメニューでアカウント名をクリックし、「**アカウント設定**」を選択します。「Partner Connect Trial」プロジェクトを選択し、概要テーブルで「**snowflake**」を選択します。「**編集**」を選択し、「**データベース**」フィールドを「analytics」に、「**ウェアハウス**」フィールドを「transforming」に更新します。

<Lightbox src="/img/snowflake_tutorial/dbt_cloud_snowflake_project_overview.png" title="dbt Cloud - Snowflake Project Overview" />

<Lightbox src="/img/snowflake_tutorial/dbt_cloud_update_database_and_warehouse.png" title="dbt Cloud - Update Database and Warehouse" />

</TabItem>
<TabItem value="manual-connect" label="Connect manually">


1. dbt Cloud で新しいプロジェクトを作成します。左側のメニューでアカウント名をクリックして [**アカウント設定**] に移動し、[**+ 新しいプロジェクト**] をクリックします。
2. プロジェクト名を入力し、[**続行**] をクリックします。
3. ウェアハウスの場合は、[**Snowflake**] をクリックし、[**次へ**] をクリックして接続を設定します。

    <Lightbox src="/img/snowflake_tutorial/dbt_cloud_setup_snowflake_connection_start.png" title="dbt Cloud - Choose Snowflake Connection" />

4. Snowflake の **設定** を入力します。
    * **アカウント** &mdash; Snowflake トライアルアカウントの URL から `snowflakecomputing.com` を削除して、アカウントを見つけます。アカウント情報の順序は、Snowflake のバージョンによって異なります。たとえば、Snowflake の Classic コンソールの URL は `oq65696.west-us-2.azure.snowflakecomputing.com` のようになります。AppUI または Snowsight の URL は `snowflakecomputing.com/west-us-2.azure/oq65696` のようになります。どちらの例でも、アカウントは `oq65696.west-us-2.azure` になります。詳細については、Snowflake ドキュメントの [アカウント識別子](https://docs.snowflake.com/en/user-guide/admin-account-identifier.html) を参照してください。

        <Snippet path="snowflake-acct-name" />
    
    * **ロール** - 今は空白のままにしておきます。後でデフォルトのSnowflakeロールに更新できます。
    * **データベース** - `analytics`。これは、dbtに分析データベースに新しいモデルを作成するように指示します。
    * **ウェアハウス** - `transforming`。これは、dbtに、先ほど作成した変換ウェアハウスを使用するように指示します。

    <Lightbox src="/img/snowflake_tutorial/dbt_cloud_snowflake_account_settings.png" title="dbt Cloud - Snowflake Account Settings" />

5. Snowflake の**開発認証情報**を入力します:
    * **ユーザー名** - Snowflake 用に作成したユーザー名です。ユーザー名はメールアドレスではなく、通常は名と姓を組み合わせた単語です。
    * **パスワード** - Snowflake アカウント作成時に設定したパスワードです。
    * **スキーマ** - スキーマ名は自動作成されています。慣例により、これは `dbt_<first-initial><last-name>` です。これは開発環境に直接接続されたスキーマであり、Cloud IDE 内で dbt を実行するときにモデルが構築される場所です。
    * **ターゲット名** - デフォルトのままにします。
    * **スレッド** - 4 のままにします。これは、dbt Cloud がモデルを同時に構築するために行う同時接続の数です。

    <Lightbox src="/img/snowflake_tutorial/dbt_cloud_snowflake_development_credentials.png" title="dbt Cloud - Snowflake Development Credentials" />

6. **「接続テスト」** をクリックします。これにより、dbt Cloud が Snowflake アカウントにアクセスできることが確認されます。
7. 接続テストが成功した場合は、**「次へ」** をクリックします。失敗した場合は、Snowflake の設定と認証情報を確認する必要がある場合があります。
</TabItem>
</Tabs>

## dbt Cloud プロジェクトのセットアップ

このセクションでは、dbt Cloud 管理リポジトリをセットアップし、dbt プロジェクトを初期化して開発を開始します。

### dbt Cloud マネージドリポジトリを設定する
Partner Connect をご利用の場合は、[マネージドリポジトリ](/docs/cloud/git/managed-repository) が提供されるため、[dbt プロジェクトの初期化](#dbt プロジェクトを初期化して開発を開始) までスキップできます。それ以外の場合は、リポジトリ接続を作成する必要があります。

<Snippet path="tutorial-managed-repo" />

### dbt プロジェクトを初期化する
このガイドでは、[dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用して dbt プロジェクトを開発し、指標を定義し、[MetricFlow コマンド](/docs/build/metricflow-commands) を使用して指標をクエリおよびプレビューすることを前提としています。

リポジトリの設定が完了したら、IDE を使用してプロジェクトを初期化し、dbt Cloud で開発を開始できます。

1. **[dbt Cloud IDE で開発を開始]** をクリックします。Git 接続の確立、リポジトリのクローン作成、ウェアハウスへの接続テストが行​​われるため、プロジェクトの初回起動には数分かかる場合があります。
2. 左側のファイルツリーの上にある [**プロジェクトの初期化]** をクリックします。これにより、サンプルモデルを含むフォルダ構造が構築されます。
3. [**コミットして同期]** をクリックして、最初のコミットを実行します。コミットメッセージには「initial commit」を使用します。これにより、マネージドリポジトリへの最初のコミットが作成され、新しい dbt コードを追加できるブランチが開きます。
4. これで、ウェアハウスから直接データをクエリし、`dbt run` を実行できるようになりました。今すぐ試してみましょう。
  - **ファイルエクスプローラー** で models/examples フォルダを削除します。
  - 「**+ 新しいファイルを作成**」をクリックし、次のクエリを新しいファイルに追加して、「**名前を付けて保存**」をクリックして新しいファイルを保存します。 
  ```sql
  select * from raw.jaffle_shop.customers
  ```
  - 下部のコマンドラインバーに「dbt run」と入力し、Enterキーを押します。「dbt run が成功しました」というメッセージが表示されます。

## dbt プロジェクトをビルドする
次のステップは、プロジェクトをビルドすることです。これには、ソース、ステージングモデル、ビジネス定義エンティティ、パッケージをプロジェクトに追加することが含まれます。

### ソースを追加する

dbt の [ソース](/docs/build/sources) は、変換する生データテーブルです。ソース定義を整理することで、データの出所を文書化できます。また、プロジェクトと変換の信頼性、構造、理解度が向上します。

dbt Cloud IDE でファイルを操作するには、次の 2 つの方法があります:

- **新しいブランチを作成する（推奨）** - 変更を編集してコミットするための新しいブランチを作成します。左側のサイドバーの [**バージョン管理**] に移動し、[**ブランチを作成**] をクリックします。
- **保護されたプライマリブランチで編集する** - ファイルの編集、フォーマット、または lint を実行し、プライマリ Git ブランチで直接 dbt コマンドを実行する場合は、このオプションを使用します。dbt Cloud IDE は保護されたブランチへのコミットを防ぐため、変更を新しいブランチにコミットするように求められます。

新しいブランチに「build-project」という名前を付けます。

1. 「models」ディレクトリにマウスを移動し、3 点メニュー (**...**) をクリックして、[**ファイルを作成**] を選択します。
2. ファイルに「staging/jaffle_shop/src_jaffle_shop.yml」という名前を付け、[**作成**] をクリックします。
3. 次のテキストをファイルにコピーし、[**保存**] をクリックします。

<File name='models/staging/jaffle_shop/src_jaffle_shop.yml'>

```yaml
version: 2

sources:
 - name: jaffle_shop
   database: raw
   schema: jaffle_shop
   tables:
     - name: customers
     - name: orders
```

</File>

:::tip
ソースファイルでは、**モデル生成** ボタンを使用して、各ソースに対して新しいモデルファイルを作成することもできます。これにより、`models` ディレクトリに指定されたソース名の新しいファイルが作成され、ソース定義のSQLコードが埋め込まれます。
:::

4. `models` ディレクトリにマウスを移動し、3 つのドットのメニュー (**...**) をクリックして、**ファイルの作成** を選択します。
5. ファイルに「staging/stripe/src_stripe.yml」という名前を付け、**作成** をクリックします。
6. 次のテキストをファイルにコピーし、**保存** をクリックします。

<File name='models/staging/stripe/src_stripe.yml'>

```yaml
version: 2

sources:
 - name: stripe
   database: raw
   schema: stripe
   tables:
     - name: payment
```
</File>

### ステージングモデルの追加
[ステージングモデル](/best-practices/how-we-structure/2-staging)は、dbtにおける最初の変換ステップです。生データをクレンジングして準備し、より複雑な変換や分析に備えます。以下の手順に従って、ステージングモデルをプロジェクトに追加してください。

1. `jaffle_shop` サブディレクトリに、`stg_customers.sql` ファイルを作成します。または、[**モデルの生成**] ボタンを使用して、ソースごとに新しいモデルファイルを作成することもできます。
2. 次のクエリをファイルにコピーし、[**保存**] をクリックします。

<File name='models/staging/jaffle_shop/stg_customers.sql'>

```sql
  select
   id as customer_id,
   first_name,
   last_name
from {{ source('jaffle_shop', 'customers') }}
```

</File>

3. 同じ「jaffle_shop」サブディレクトリに「stg_orders.sql」ファイルを作成します。
4. 次のクエリをファイルにコピーし、「**保存**」をクリックします。

<File name='models/staging/jaffle_shop/stg_orders.sql'>

```sql
  select
    id as order_id,
    user_id as customer_id,
    order_date,
    status
  from {{ source('jaffle_shop', 'orders') }}
```

</File>

5. `stripe` サブディレクトリに、ファイル `stg_payments.sql` を作成します。
6. 次のクエリをファイルにコピーし、**[保存]** をクリックします。

<File name='models/staging/stripe/stg_payments.sql'>

```sql
select
   id as payment_id,
   orderid as order_id,
   paymentmethod as payment_method,
   status,
   -- amount is stored in cents, convert it to dollars
   amount / 100 as amount,
   created as created_at


from {{ source('stripe', 'payment') }}
```

</File>

7. 画面下部のコマンドプロンプトに「dbt run」と入力します。実行が成功し、3つのモデルが表示されます。

### ビジネス定義エンティティの追加

このフェーズでは、[dbt プロジェクトのエンティティレイヤーまたはコンセプトレイヤーとして機能するモデル](/best-practices/how-we-structure/4-marts)を作成し、レポート作成と分析のためのデータ準備を行います。また、dbt の機能を拡張する[パッケージ](/docs/build/packages)と[MetricFlow タイムスパイン](/docs/build/metricflow-time-spine)の追加も含まれます。

このフェーズは[marts レイヤー](/best-practices/how-we-structure/1-guide-overview#guide-structure-overview)であり、モジュール化された要素を統合して、組織が重視するエンティティの幅広く豊富なビジョンを構築します。

1. ファイル `models/marts/fct_orders.sql` を作成します。
2. 次のクエリをファイルにコピーし、[**保存**] をクリックします。

<File name='models/marts/fct_orders.sql'>

```sql
with orders as  (
   select * from {{ ref('stg_orders' )}}
),


payments as (
   select * from {{ ref('stg_payments') }}
),


order_payments as (
   select
       order_id,
       sum(case when status = 'success' then amount end) as amount


   from payments
   group by 1
),


final as (


   select
       orders.order_id,
       orders.customer_id,
       orders.order_date,
       coalesce(order_payments.amount, 0) as amount


   from orders
   left join order_payments using (order_id)
)


select * from final

```

</File>

3. `models/marts` ディレクトリに、ファイル `dim_customers.sql` を作成します。
4. 次のクエリをファイルにコピーし、**[保存]** をクリックします。

<File name='models/marts/dim_customers.sql'>

```sql
with customers as (
   select * from {{ ref('stg_customers')}}
),
orders as (
   select * from {{ ref('fct_orders')}}
),
customer_orders as (
   select
       customer_id,
       min(order_date) as first_order_date,
       max(order_date) as most_recent_order_date,
       count(order_id) as number_of_orders,
       sum(amount) as lifetime_value
   from orders
   group by 1
),
final as (
   select
       customers.customer_id,
       customers.first_name,
       customers.last_name,
       customer_orders.first_order_date,
       customer_orders.most_recent_order_date,
       coalesce(customer_orders.number_of_orders, 0) as number_of_orders,
       customer_orders.lifetime_value
   from customers
   left join customer_orders using (customer_id)
)
select * from final
```

</File>

5. メインディレクトリに、`packages.yml` ファイルを作成します。
6. 次のテキストをファイルにコピーし、[**保存**] をクリックします。

<File name='packages.yml'>

```sql
packages:
 - package: dbt-labs/dbt_utils
   version: 1.1.1
```

</File>

7. `models` ディレクトリ内のメインディレクトリに、ファイル `metrics/metricflow_time_spine.sql` を作成します。
8. 次のクエリをファイルにコピーし、[**保存**] をクリックします。

<File name='models/metrics/metricflow_time_spine.sql'>

```sql
{{
   config(
       materialized = 'table',
   )
}}
with days as (
   {{
       dbt_utils.date_spine(
           'day',
           "to_date('01/01/2000','mm/dd/yyyy')",
           "to_date('01/01/2027','mm/dd/yyyy')"
       )
   }}
),
final as (
   select cast(date_day as date) as date_day
   from days
)
select * from final

```

</File>

9. 画面下部のコマンドプロンプトに「dbt run」と入力します。実行成功のメッセージが表示され、実行の詳細画面でdbtが5つのモデルを正常に構築したことを確認できます。

## セマンティックモデルの作成

このセクションでは、[セマンティックモデル](https://docs.getdbt.com/guides/sl-snowflake-qs?step=6#about-semantic-models)、[そのコンポーネント](https://docs.getdbt.com/guides/sl-snowflake-qs?step=6#semantic-model-components)、および[タイムスパインの設定方法](https://docs.getdbt.com/guides/sl-snowflake-qs?step=6#configure-a-time-spine)について学習します。


### セマンティックモデルについて

[セマンティックモデル](/docs/build/semantic-models)には、MetricFlowがメトリック定義のクエリを構築するために必要な多くのオブジェクトタイプ（エンティティ、メジャー、ディメンションなど）が含まれています。

- 各セマンティックモデルは、dbt SQL/Pythonモデルと1対1で対応します。
- 各セマンティックモデルには、（最大で）1つのプライマリエンティティまたはナチュラルエンティティが含まれます。
- 各セマンティックモデルには、他のエンティティへの接続に使用される0個、1個、または複数の外部エンティティまたは一意のエンティティが含まれます。
- 各セマンティックモデルには、ディメンション、メジャー、メトリックが含まれる場合もあります。これらは、下流のBIツールに実際に入力され、クエリされるものです。

次の手順では、セマンティックモデルを使用して、注文に関連するデータの解釈方法を定義できます。これには、エンティティ（データの結合キーとして機能するID列など）、ディメンション（データのグループ化またはフィルタリング用）、メジャー（データ集計用）が含まれます。

1. `metrics` サブディレクトリに、新しいファイル `fct_orders.yml` を作成します。

:::tip 
すべてのセマンティックモデルとメトリクスは、[`model-paths`](/reference/project-configs/model-paths) で定義されたディレクトリ（または `models/semantic_models/` のようなそのサブディレクトリ）に保存してください。このパス以外に保存すると、空の `semantic_manifest.json` ファイルが作成され、セマンティックモデルやメトリクスが認識されなくなります。
:::

2. 新しく作成したファイルに次のコードを追加します:

<File name='models/metrics/fct_orders.yml'>

```yaml
semantic_models:
  - name: orders
    defaults:
      agg_time_dimension: order_date
    description: |
      Order fact table. This table’s grain is one row per order.
    model: ref('fct_orders')
```

</File>

### セマンティックモデルの構成要素

以下のセクションでは、[ディメンション](/docs/build/dimensions)、[エンティティ](/docs/build/entities)、[メジャー](/docs/build/measures) について詳しく説明し、それぞれがセマンティックモデルでどのような役割を果たすかを示します。

- [エンティティ](#entities) は、異なるテーブルのデータをリンクする一意の識別子（ID 列など）として機能します。
- [ディメンション](#dimensions) は、データを分類およびフィルタリングして、整理を容易にします。
- [メジャー](#measures) は、データを計算し、集計を通じて貴重な洞察を提供します。


### エンティティ

[エンティティ](/docs/build/semantic-models#entities)は、ビジネスにおける現実世界の概念であり、セマンティックモデルのバックボーンとして機能します。これらは、セマンティックモデルにおいてID列（`order_id`など）として機能します。また、他のセマンティックモデルへの結合キーとして機能します。

`fct_orders.yml`セマンティックモデルファイルにエンティティを追加します。

<File name='models/metrics/fct_orders.yml'>

```yaml
semantic_models:
  - name: orders
    defaults:
      agg_time_dimension: order_date
    description: |
      Order fact table. This table’s grain is one row per order.
    model: ref('fct_orders')
    # Newly added
    entities: 
      - name: order_id
        type: primary
      - name: customer
        expr: customer_id
        type: foreign
```

</File>

### ディメンション

[ディメンション](/docs/build/semantic-models#entities)は、カテゴリや時間に基づいて情報をグループ化またはフィルタリングする方法です。

`fct_orders.yml` セマンティックモデルファイルにディメンションを追加します。

<File name='models/metrics/fct_orders.yml'>

```yaml
semantic_models:
  - name: orders
    defaults:
      agg_time_dimension: order_date
    description: |
      Order fact table. This table’s grain is one row per order.
    model: ref('fct_orders')
    entities:
      - name: order_id
        type: primary
      - name: customer
        expr: customer_id
        type: foreign
    # Newly added
    dimensions:   
      - name: order_date
        type: time
        type_params:
          time_granularity: day
```

</File>

### メジャー

[メジャー](/docs/build/semantic-models#measures) は、モデル内の列に対して実行される集計です。多くの場合、メジャー自体を最終的なメトリクスとして使用します。また、メジャーはより複雑なメトリクスの構成要素としても機能します。

`fct_orders.yml` セマンティックモデルファイルにメジャーを追加します。

<File name='models/metrics/fct_orders.yml'>

```yaml
semantic_models:
  - name: orders
    defaults:
      agg_time_dimension: order_date
    description: |
      Order fact table. This table’s grain is one row per order.
    model: ref('fct_orders')
    entities:
      - name: order_id
        type: primary
      - name: customer
        expr: customer_id
        type: foreign
    dimensions:
      - name: order_date
        type: time
        type_params:
          time_granularity: day
    # Newly added      
    measures:   
      - name: order_total
        description: The total amount for each order including taxes.
        agg: sum
        expr: amount
      - name: order_count
        expr: 1
        agg: sum
      - name: customers_with_orders
        description: Distinct count of customers placing orders
        agg: count_distinct
        expr: customer_id
      - name: order_value_p99 ## The 99th percentile order value
        expr: amount
        agg: percentile
        agg_params:
          percentile: 0.99
          use_discrete_percentile: True
          use_approximate_percentile: False
```

</File>

### タイムスパインの設定

正確な時間ベースの集計を行うには、[タイムスパイン](/docs/build/metricflow-time-spine)を設定する必要があります。タイムスパインを使用すると、さまざまな時間粒度で正確なメトリック計算を行うことができます。

1. メトリックに必要な粒度（日次、時間別など）で、プロジェクトにタイムスパインモデルを追加します。
2. YAMLファイルで各タイムスパインを設定し、MetricFlowが列を認識して使用する方法を定義します。[YAMLでのタイムスパインの設定](/docs/build/metricflow-time-spine#configuring-time-spine-in-yaml)ドキュメントの手順に従ってください。

詳細な手順については、[MetricFlow タイムスパインガイド](/guides/mf-time-spine?step=1)を参照してください。

## メトリクスを定義し、2つ目のセマンティックモデルを追加する

このセクションでは、[メトリクスを定義](#define-metrics)し、[2つ目のセマンティックモデルをプロジェクトに追加](#add-second-semantic-model-to-your-project)します。

### 指標の定義

[指標](/docs/build/metrics-overview) は、ビジネスユーザーがビジネスパフォーマンスを測定するための言語です。これは、ウェアハウス内の列を集計したもので、ディメンションカットによって拡張されます。

設定できる指標には、以下の種類があります。

- [コンバージョン指標](/docs/build/conversion) - 設定された期間内に、エンティティのベースイベントとそれに続くコンバージョンイベントがいつ発生したかを追跡します。
- [累積指標](/docs/build/metrics-overview#cumulative-metrics) - 指定された期間の指標を集計します。期間が指定されていない場合、その期間は記録された期間全体にわたって指標を累積します。累積指標を追加する前に、タイムスパインモデルを作成する必要があることに注意してください。
- [派生指標](/docs/build/metrics-overview#derived-metrics) - 指標に基づいて計算を行うことができます。
- [シンプルメトリクス](/docs/build/metrics-overview#simple-metrics) - 追加のメトリクスを使用せずに、単一のメトリクスを直接参照します。
- [比率メトリクス](/docs/build/metrics-overview#ratio-metrics) - 分子メトリクスと分母メトリクスを使用します。制約文字列は、分子と分母の両方に適用することも、分子または分母に個別に適用することもできます。

セマンティックモデルを作成したら、作成したメジャーを参照してメトリクスを作成します。

1. `fct_orders.yml` セマンティックモデルファイルにメトリクスを追加します。

:::tip 
すべてのセマンティックモデルとメトリクスは、[`model-paths`](/reference/project-configs/model-paths) で定義されたディレクトリ（または `models/semantic_models/` のようなそのサブディレクトリ）に保存してください。このパス以外に保存すると、空の `semantic_manifest.json` ファイルが作成され、セマンティックモデルやメトリクスが認識されなくなります。
:::

<File name='models/metrics/fct_orders.yml'>

```yaml
semantic_models:
  - name: orders
    defaults:
      agg_time_dimension: order_date
    description: |
      Order fact table. This table’s grain is one row per order
    model: ref('fct_orders')
    entities:
      - name: order_id
        type: primary
      - name: customer
        expr: customer_id
        type: foreign
    dimensions:
      - name: order_date
        type: time
        type_params:
          time_granularity: day
    measures:
      - name: order_total
        description: The total amount for each order including taxes.
        agg: sum
        expr: amount
      - name: order_count
        expr: 1
        agg: sum
      - name: customers_with_orders
        description: Distinct count of customers placing orders
        agg: count_distinct
        expr: customer_id
      - name: order_value_p99
        expr: amount
        agg: percentile
        agg_params:
          percentile: 0.99
          use_discrete_percentile: True
          use_approximate_percentile: False
# Newly added          
metrics: 
  # Simple type metrics
  - name: "order_total"
    description: "Sum of orders value"
    type: simple
    label: "order_total"
    type_params:
      measure:
        name: order_total
  - name: "order_count"
    description: "number of orders"
    type: simple
    label: "order_count"
    type_params:
      measure:
        name: order_count
  - name: large_orders
    description: "Count of orders with order total over 20."
    type: simple
    label: "Large Orders"
    type_params:
      measure:
        name: order_count
    filter: |
      {{ Metric('order_total', group_by=['order_id']) }} >=  20
  # Ratio type metric
  - name: "avg_order_value"
    label: "avg_order_value"
    description: "average value of each order"
    type: ratio
    type_params:
      numerator: 
        name: order_total
      denominator: 
        name: order_count
  # Cumulative type metrics
  - name: "cumulative_order_amount_mtd"
    label: "cumulative_order_amount_mtd"
    description: "The month to date value of all orders"
    type: cumulative
    type_params:
      measure:
        name: order_total
      grain_to_date: month
  # Derived metric
  - name: "pct_of_orders_that_are_large"
    label: "pct_of_orders_that_are_large"
    description: "percent of orders that are large"
    type: derived
    type_params:
      expr: large_orders/order_count
      metrics:
        - name: large_orders
        - name: order_count
```

</File>

### プロジェクトに2つ目のセマンティックモデルを追加しましょう

おめでとうございます！最初のセマンティックモデルの構築に成功しました！エンティティ、ディメンション、メジャー、メトリックなど、必要な要素がすべて揃っています。

もう1つのマートモデル（`dim_customers.yml` など）にセマンティックモデルを追加して、プロジェクトの分析機能を拡張しましょう。

注文モデルの設定後：

1. `metrics` サブディレクトリに、`dim_customers.yml` ファイルを作成します。
2. 次のクエリをファイルにコピーし、**[保存]** をクリックします。

<File name='models/metrics/dim_customers.yml'>

```yaml
semantic_models:
  - name: customers
    defaults:
      agg_time_dimension: most_recent_order_date
    description: |
      semantic model for dim_customers
    model: ref('dim_customers')
    entities:
      - name: customer
        expr: customer_id
        type: primary
    dimensions:
      - name: customer_name
        type: categorical
        expr: first_name
      - name: first_order_date
        type: time
        type_params:
          time_granularity: day
      - name: most_recent_order_date
        type: time
        type_params:
          time_granularity: day
    measures:
      - name: count_lifetime_orders
        description: Total count of orders per customer.
        agg: sum
        expr: number_of_orders
      - name: lifetime_spend
        agg: sum
        expr: lifetime_value
        description: Gross customer lifetime spend inclusive of taxes.
      - name: customers
        expr: customer_id
        agg: count_distinct

metrics:
  - name: "customers_with_orders"
    label: "customers_with_orders"
    description: "Unique count of customers placing orders"
    type: simple
    type_params:
      measure:
        name: customers
```

</File>

このセマンティックモデルは、シンプルな指標を用いて顧客指標に焦点を当て、氏名、タイプ、注文日といった顧客ディメンションを重視します。顧客行動、生涯価値、注文パターンを独自に分析します。

## Test and query metrics

<!-- The below snippets (or reusables) can be found in the following file locations in the docs code repository) 

https://github.com/dbt-labs/docs.getdbt.com/blob/current/website/snippets/_sl-test-and-query-metrics.md
-->

<TestQuery />

## Run a production job

<!-- The below snippets (or reusables) can be found in the following file locations in the docs code repository) 

https://github.com/dbt-labs/docs.getdbt.com/blob/current/website/snippets/_sl-run-prod-job.md
-->

<RunProdJob/>


## dbt セマンティックレイヤーの設定

このセクションでは、dbt セマンティックレイヤーの設定、認証情報の追加、サービストークンの作成方法を学習します。このセクションでは、以下のトピックについて説明します。

- [環境の選択](#1-select-environment)
- [認証情報の追加とサービストークンの作成](#2-add-a-credential-and-create-service-tokens)
- [接続の詳細の表示](#3-view-connection-detail)
- [認証情報の追加](#4-add-more-credentials)
- [構成の削除](#delete-configuration)

<!-- The below snippets (or reusables) can be found in the following file locations in the docs code repository) 

https://github.com/dbt-labs/docs.getdbt.com/blob/current/website/snippets/_new-sl-setup.md
-->

<SlSetUp/>

## セマンティックレイヤーへのクエリ

このページでは、以下の統合機能に接続してメトリクスのクエリを実行する方法について説明します。

- [Google スプレッドシートに接続してクエリを実行する](#connect-and-query-with-google-sheets)
- [Hex に接続してクエリを実行する](#connect-and-query-with-hex)

dbt セマンティックレイヤーを使用すると、Google スプレッドシート、Hex、Tableau などのさまざまなツールに接続してメトリクスのクエリを実行できます。

[ファーストクラス統合](/docs/cloud-integrations/avail-sl-integrations)、[セマンティックレイヤー API](/docs/dbt-cloud-apis/sl-api-overview)、[エクスポート](/docs/use-dbt-semantic-layer/exports)などのツールを使用してメトリクスのクエリを実行し、データプラットフォーム内のメトリクスとディメンションのテーブルを公開して、PowerBI などのツールとのカスタム統合を作成できます。

 ### Connect and query with Google Sheets

<!-- The below snippets (or reusables) can be found in the following file locations in the docs code repository) 

https://github.com/dbt-labs/docs.getdbt.com/blob/current/website/snippets/_sl-connect-and-query-api.md
-->

<ConnectQueryAPI/>

### Hex に接続してクエリを実行する
このセクションでは、Hex 統合を使用して Hex でメトリクスをクエリする方法について説明します。接続方法に応じて適切なタブを選択してください。

<Tabs>
<TabItem value="partner-connect" label="Query Semantic Layer with Hex" default>

1. [Hex ログインページ](https://app.hex.tech/login) に移動します。
2. ログインするか、アカウントを作成します（まだアカウントをお持ちでない場合）。
- 職場のメールアドレスまたは .edu メールアドレスで、Hex の無料トライアルアカウントを作成できます。
3. ページの左上にある **HEX** アイコンをクリックして、ホームページに移動します。
4. 次に、右上にある **+ 新しいプロジェクト** ボタンをクリックします。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_new.png" width="50%" title="Click the '+ New project' button on the top right"/>

5. 左側のメニューに移動し、**データブラウザ**を選択します。次に、**データ接続の追加**を選択します。
6. **Snowflake**をクリックします。データ接続の名前と説明を入力します。セマンティックレイヤーを使用するのに、データウェアハウスの認証情報は必要ありません。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_new_data_connection.png" width="50%" title="Select 'Data browser' and then 'Add a data connection' to connect to Snowflake."/>

7. [**統合**] の下で、dbt スイッチを右に切り替えて、dbt 統合を有効にします。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_dbt_toggle.png" width="50%" title="Click on the dbt toggle to enable the integration. "/>

8. 以下の情報を入力します。
    * dbt のバージョンとして 1.6 以上を選択します。
    * 環境 ID を入力します。
    * サービストークンを入力します。
    * **「セマンティック レイヤーを使用する」** トグルを必ずクリックしてください。これにより、すべてのクエリが dbt を経由するようになります。
    * 右下にある **接続を作成** をクリックします。
9. 次の画像に示すメニューの **詳細** にマウスを移動し、**dbt セマンティック レイヤー** を選択します。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_make_sl_cell.png" width="90%" title="Hover over 'More' on the menu and select 'dbt Semantic Layer'."/>

10. これで、Hex を使ってメトリクスをクエリできるようになりました！ぜひお試しください。
    - 新しいセルを作成し、メトリクスを選択します。
    - 1 つ以上のディメンションでフィルタリングします。
    - ビジュアライゼーションを作成します。

</TabItem>
<TabItem value="manual-connect" label="Getting started with the Semantic Layer workshop">

1. ワークショップのチャットで提供されたリンクをクリックしてください。
    - すぐに表示されない場合は、チャットの**ピン留めされたメッセージ**セクションをご覧ください。
2. 指定されたテキストボックスにメールアドレスを入力してください。次に、**SQLとPython**を選択すると、Hexのホーム画面に移動します。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/welcome_to_hex.png" width="70%" title="The 'Welcome to Hex' homepage."/>

3. 次に、左上隅にある紫色のHexボタンをクリックします。
4. 左側のメニューで**Collections**ボタンをクリックします。
5. **Semantic Layer Workshop**コレクションを選択します。
6. **Getting started with the dbt Semantic Layer**プロジェクトコレクションをクリックします。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_collections.png" width="80%" title="Click 'Collections' to select the 'Semantic Layer Workshop' collection."/>

7. このHexノートブックを編集するには、プロジェクトのドロップダウンメニュー（次の画像を参照）から**複製**ボタンをクリックします。これにより、所有しているHexノートブックの新しいコピーが作成されます。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_duplicate.png" width="80%" title="Click the 'Duplicate' button from the project dropdown menu to create a Hex notebook copy."/>

8. 見つけやすくするために、Hex プロジェクトのコピーの名前を自分の名前を含むように変更します。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_rename.png" width="60%" title="Rename your Hex project to include your name."/>

9. これで、Hex を使ってメトリクスをクエリできるようになりました。以下のサンプルクエリを実際に試してみてください。

    - 最初のセルには、`order_total` メトリクスの推移を示す表が表示されています。この表に `order_count` メトリクスを追加してみましょう。
    - 2 番目のセルには、`order_total` メトリクスの推移を示す折れ線グラフが表示されています。グラフを操作してみましょう。**時間単位** ドロップダウンメニューを使って、時間粒度を変更してみましょう。
    - ノートブックの次の表「Example_query_2」には、特定の日に初めて注文した顧客の数が表示されています。新しいグラフセ​​ルを作成します。`first_ordered_at` と `customers` の折れ線グラフを作成し、毎日の新規顧客数が時間とともにどのように変化しているかを確認します。
    - 新しいセマンティックレイヤーセルを作成し、1 つ以上のメトリクスを選択します。1 つ以上のディメンションでメトリクスをフィルタリングします。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/hex_make_sl_cell.png" width="90%" title="Query metrics using Hex "/>

</TabItem>
</Tabs>

## 次は何？

<ConfettiTrigger>

包括的なdbtセマンティックレイヤーガイドの完成、おめでとうございます🎉！dbtセマンティックレイヤーとは何か、その目的、そしてプロジェクトでいつ使用するのかについて、明確に理解していただけたかと思います。

学習内容：

- Snowflake環境とdbt Cloudの設定（ワークシートの作成とデータのロードを含む）。
- dbt CloudをSnowflakeに接続して構成する。
- メトリクスとセマンティックレイヤーに焦点を当て、dbt Cloudプロジェクトの構築、テスト、管理を行う。
- 利用可能な統合機能を使用して、本番環境ジョブを実行し、メトリクスをクエリする。

次のステップでは、独自のメトリックの定義を開始し、[エクスポート](/docs/use-dbt-semantic-layer/exports)、[null 値の入力](/docs/build/advanced-topics)、[セマンティック レイヤーを使用した dbt Mesh の実装](/docs/use-dbt-semantic-layer/sl-faqs#how-can-i-implement-dbt-mesh-with-the-dbt-semantic-layer) などの追加の構成オプションについて学習できます。

引き続き学習を進める上で役立つ追加リソースをご紹介します。

- [dbt セマンティックレイヤーに関するよくある質問](/docs/use-dbt-semantic-layer/sl-faqs)
- [利用可能な統合](/docs/cloud-integrations/avail-sl-integrations)
- [MetricFlow でメトリクスを定義およびクエリする方法](https://www.loom.com/share/60a76f6034b0441788d73638808e92ac?sid=861a94ac-25eb-4fd8-a310-58e159950f5a) のデモ
- [ライブデモに参加する](https://www.getdbt.com/resources/webinars/dbt-cloud-demos-with-experts)

</ConfettiTrigger>
