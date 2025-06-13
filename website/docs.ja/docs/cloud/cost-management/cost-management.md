---
title: "About cost management in dbt"
description: "Manage your data warehouse costs in dbt"
sidebar_label: About cost management
---

# dbt のコスト管理について <Lifecycle status='preview,managed,managed_plus' />

<Constant name="cloud" /> のコスト管理ダッシュボードでは、dbt プロジェクトがデータウェアハウスのコストにどのような影響を与えているかについて、貴重な洞察が得られます。モデル、テスト、スナップショット、その他のリソースなどの機能が時間の経過とともにコストにどのように影響するかを視覚化することで、ウェアハウスの支出を最適化し、対策を講じ、関係者に報告し、開発ワークフローを最適化できるようになります。

現在、Snowflake のみがサポートされています。

このドキュメントでは、Snowflake と <Constant name="cloud" /> の設定、そしてコスト管理ダッシュボードを使用して洞察を確認する方法について説明します。

## 前提条件

コスト管理ツールを構成するには、以下の要件を満たす必要があります。

- <Constant name="cloud" /> で接続を構成するための適切な [権限セット](/docs/cloud/manage-access/enterprise-permissions) (アカウント管理者やプロジェクト作成者など)。
- ユーザーを作成し、データベースアクセスを割り当てるための、Snowflake での適切な [権限](https://docs.snowflake.com/en/user-guide/security-access-control-privileges)。
- サポートされているデータウェアハウス。注: 現時点では Snowflake のみがサポートされています。今後、他のウェアハウスもサポートされる予定です。
- [Enterprise または Enterprise+ プラン](https://www.getdbt.com/pricing) の <Constant name="cloud" /> アカウント。


## Snowflake での設定

コスト管理ツールで監視する Snowflake アカウントごとに、メタデータ認証情報を設定する必要があります。Snowflake で適切なアクセスを設定するには、以下の手順に従います。

1. Snowflake アカウントで、既存または新規（推奨）のサービスユーザーを特定します。より柔軟なカスタマイズのため、このサービスには新しいユーザー（例：dbt_cost_user）を設定することをお勧めします。
2. ユーザーに [`ORGANIZATION_USAGE`](https://docs.snowflake.com/en/sql-reference/organization-usage) および [`ACCOUNT_USAGE`](https://docs.snowflake.com/en/sql-reference/account-usage) スキーマへの `read` 権限を付与します。
    - (オプション) 必要に応じて、次のアクセス権が割り当てられた [Snowflake データベース ロール](https://docs.snowflake.com/en/sql-reference/account-usage#enabling-other-roles-to-use-schemas-in-the-snowflake-database) を使用して、ウェアハウス内の特定のテーブルに範囲を絞り込むことができます。
        - `ACCOUNT_USAGE.QUERY_HISTORY`
        - `ACCOUNT_USAGE.QUERY_ATTRIBUTION_HISTORY`
        - `ACCOUNT_USAGE.ACCESS_HISTORY`
        - `ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY`
        - `ORGANIZATION_USAGE.USAGE_IN_CURRENCY_DAILY`


SQL を使用してユーザー `dbt_cost_user` とロール `dbt_cost_management` を作成し、特定のテーブルに対する必要な権限を割り当てるには、次の例のようなものを実行します:

```sql

CREATE USER dbt_cost_user
  PASSWORD = 'A_SECURE_PASSWORD'
  DEFAULT_ROLE = dbt_cost_management
  MUST_CHANGE_PASSWORD = FALSE;

CREATE ROLE dbt_cost_management;

GRANT ROLE dbt_cost_management TO USER dbt_cost_user;

GRANT USAGE ON DATABASE SNOWFLAKE TO ROLE dbt_cost_management;
GRANT USAGE ON SCHEMA SNOWFLAKE.ACCOUNT_USAGE TO ROLE dbt_cost_management;

GRANT SELECT ON VIEW SNOWFLAKE.ACCOUNT_USAGE.QUERY_HISTORY TO ROLE dbt_cost_management;
GRANT SELECT ON VIEW SNOWFLAKE.ACCOUNT_USAGE.QUERY_ATTRIBUTION_HISTORY TO ROLE dbt_cost_management;
GRANT SELECT ON VIEW SNOWFLAKE.ACCOUNT_USAGE.ACCESS_HISTORY TO ROLE dbt_cost_management;
GRANT SELECT ON VIEW SNOWFLAKE.ACCOUNT_USAGE.WAREHOUSE_METERING_HISTORY TO ROLE dbt_cost_management;

GRANT USAGE ON SCHEMA SNOWFLAKE.ORGANIZATION_USAGE TO ROLE dbt_cost_management;
GRANT SELECT ON VIEW SNOWFLAKE.ORGANIZATION_USAGE.USAGE_IN_CURRENCY_DAILY TO ROLE dbt_cost_management;

```

より広範なアカウント全体のアクセスを実現するには、ユーザーに `IMPORTED PRIVILEGES` を割り当てることができます:

```sql

CREATE USER dbt_cost_user
  PASSWORD = 'A_SECURE_PASSWORD'
  DEFAULT_ROLE = dbt_cost_management
  MUST_CHANGE_PASSWORD = FALSE;

CREATE ROLE dbt_cost_management;
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO ROLE dbt_cost_management;
GRANT ROLE dbt_cost_management TO USER dbt_cost_user;

```

必要に応じて、<Constant name="cloud" /> を使用して、ユーザー名とパスワードの代わりにキーペア認証を使用するようにユーザーを設定することもできます。

監視する各 Snowflake ウェアハウスでユーザー作成プロセスを繰り返す必要があります。

ユーザーを作成し、適切な権限を割り当てたら、<Constant name="cloud" /> で接続を設定します。

## dbt での設定

コスト管理機能を構成するには、接続とユーザーコンポーネントの両方が必要です。

- **[接続の設定](#connection-setup):** データウェアハウス情報へのアクセスに使用する認証情報を設定します。ウェアハウスごとに 1 つの固有の [接続](/docs/cloud/connect-data-platform/about-connections#connection-management) のみに認証情報を設定する必要があります。

- **[ユーザーアクセスのプロビジョニング](#provision-user-access):** ダッシュボードへのアクセスを制御するために、ユーザーまたはグループに新しい権限を追加します。

### 接続設定

dbt でメタデータ接続を構成するには：

1. **Account Settings** に移動し、**Connections** をクリックします。
2. Snowflake 設定で構成したデータウェアハウスに関連付けられた接続をクリックします。**Edit** はクリックしないでください。これはより広範な設定のためであり、メタデータセクションが変更されるのを防ぎます。
3. **Platform metadata credentials** まで下にスクロールし、**Add credentials** をクリックします。
4. 適切な**Auth method**（ユーザー名とパスワード、またはキーペア）を設定し、すべてのフィールドに入力します。
5. **Features** セクションで、**Cost Management** を有効にするボックスをクリックします。

    <Lightbox src="/img/docs/dbt-cloud/cost-management/configure-metadata.png" width="60%" title="Fill out the fields with the appropriate information."/>

6. **Save** をクリックします。
7. 監視する各 <Constant name="cloud" /> ウェアハウス接続に対して、このプロセスを繰り返します。

セットアップ後、初期同期が完了し、ダッシュボードに情報が表示され始めるまで数時間かかります。

### ユーザーアクセスのプロビジョニング

ダッシュボードには機密性の高い財務情報が含まれているため、アクセス制御に役立つ 2 つの新しい権限セット「コスト管理管理者」と「コスト管理閲覧者」を導入しました。

「コスト管理閲覧者」ロールは、管理者ロールに関連付けられた昇格された権限なしでダッシュボードへの閲覧者アクセスを許可したい組織にとって特に便利です。

例えば、コストの観測性とインサイトを担当する開発者グループにダッシュボードの閲覧アクセスを許可したいとします。
1. **Account settings** に移動し、**Groups and licenses** を開きます。
2. リストからグループをクリックし、**Edit** をクリックします。
3. **Accounts and permissions** から、**Add permission** をクリックします。
4. ドロップダウンから **Cost Management Viewer** 権限を選択し、**Save** をクリックします。

<Lightbox src="/img/docs/dbt-cloud/cost-management/cost-management-viewer.png" width="60%" title="The Cost Management Viewer role assigned to a group."/>

ダッシュボードへのアクセスを許可するユーザーまたはグループにこれらの権限セットを割り当てることで、他のロールによるより広範なアクセス権限の付与を回避できます。

## コスト管理ダッシュボード

コスト管理ダッシュボードは、<Constant name="cloud" /> 内のどこからでも左側のメニューからアクセスできます。有効にすると、サイドバー上部の **Account home** 機能の下に **Cost management** オプションが表示されます。適切な権限を持たないユーザーには、このオプションは表示されません。

以下の [権限セット](/docs/cloud/manage-access/enterprise-permissions) を持つユーザーがコスト管理ダッシュボードにアクセスできます。
- アカウント管理者
- アカウント閲覧者
- **新機能:** コスト管理閲覧者
- **新機能:** コスト管理管理者

情報が同期されると、左側のメニューから [**コスト管理**] ダッシュボード オプションを選択すると、結果が表示されます。

<Lightbox src="/img/docs/dbt-cloud/cost-management/dashboard-upper.png" width="60%" title="The cost management overview."/>

<Lightbox src="/img/docs/dbt-cloud/cost-management/dashboard.png" width="60%" title="More of the cost management dashboard overview."/>

- **Last refreshed...** の日付にマウスを合わせると、構成された接続とそのステータスのリストが表示されます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/connection-status.png" width="60%" title="View your connection status."/>
- 監視する期間を調整します。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/time-period.png" width="60%" title="Adjust the period you want to view."/>

### メトリクス

ダッシュボードを操作すると、コストを表示および測定できるメトリクスがあります。ダッシュボードをフィルタリングすると、これらのメトリクスで並べ替えることができるリストビューにアクセスできます。コスト管理ダッシュボードでは、以下のメトリクスを利用できます。

- **実行クエリ:** データウェアハウス内で dbt リソース（モデルビルド、テストなど）の実行によって実行されたクエリの総数。
- **消費クエリ:** ウェアハウス内のすべての使用状況（BI/分析ツール、クエリコンソールなどを含む）における、特定のリソースに対するクエリの総数。
- **実行コスト:** dbt 実行で実行されているリソースに関連するウェアハウスの総コスト。
- **期間（リソースビューのみ）:** 期間中に dbt リソースを実行したクエリの合計期間。

これらのメトリクスでリストビューを並べ替えることで、リソースが個々の領域にどのように影響しているかを確認し、最もコストの高い領域をすばやく確認できます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/sort-by-execution-cost.png" width="60%" title="Metrics sorted by execution cost."/>
    <Lightbox src="/img/docs/dbt-cloud/cost-management/sort-by-consumption-query.png" width="60%" title="Metrics sorted by consumption queries."/>

### 概要

**Overview** ダッシュボードは最初に表示される画面です。ここでは、コストに関する一般的な情報を確認できます。

- 選択した期間における倉庫コストの傾向を表示します。グラフにマウスポインターを合わせると、期間ごとの支出の差額が表示されます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/cost-trends.png" width="60%" title="View trends over time."/>
- 上部のタイルには、以下の情報が表示されます。
    - 選択した期間の倉庫支出。
    - 実現した節約額（近日公開予定）。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/warehouse-spend.png" width="60%" title="See your total spending."/>
- 棒グラフは、プロジェクトごとにdbt実行コストの内訳を示しています。個々の棒グラフをクリックすると、詳細情報が表示されます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/project-bar.png" width="60%" title="View your spending over time by project and interact with the data to view more."/>
- 
バーまたはプロジェクトをクリックすると、**Discover**タブが表示されます。ここでは、支出に関するより詳細な情報を確認できます。

### 発見

**Discover** タブをクリックすると、コスト分析をより詳細に分析できるパネルが表示されます。これはインタラクティブなページで、プロジェクト、リソース、日付、そしてそれらのさまざまな組み合わせごとにコストを内訳できます。プロジェクト系統内の特定のメモを監視して、支出に最も影響を与えているリソースを特定し、それらのリソース固有のメタデータを表示できます。

コストデータビューをフィルタリングするための複数のオプションがあり、開始点として2つの選択肢があります。
- Resources
- Environments

    <Lightbox src="/img/docs/dbt-cloud/cost-management/filter-by-resource.png" width="60%" title="Filter the Discover view by resource types."/>

#### リソースビュー

リソースでフィルタリングすると、プロジェクトのリソースが倉庫コストにどのような影響を与えているかについて、貴重な洞察が得られます。ドロップダウンメニューを使用するか、色付きの四角をクリックして、棒グラフとリストビューにリソースの種類を追加または削除してください。

- 以下のリソースの種類で情報をフィルタリングします。
    - Model
    - Test
    - Operation
    - Snapshot
    - Seed
    - Source
- グラフビューをプロジェクトやリソースの種類でフィルタリングできます。
- リソースと関連コストの詳細な内訳を表示します。リソース名や種類でフィルタリングしたり、各列で並べ替えたりできます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/resource-type.png" width="60%" title="Filter and view detailed breakdowns of your resources."/>
- リソースをクリックすると、その系統と各ノードがコストにどの程度影響するかが表示されます。このビューからdbt Explorerでリソースを開いて、メタデータをより深く理解することもできます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/render-lineage.png" width="60%" title="View the resources lineage and monitor node costs."/>

#### 環境ビュー

環境でフィルタリングする場合、プロジェクトを選択すると、各環境タイプが倉庫コストにどのような影響を与えるかについての詳細情報が表示されます。

    <Lightbox src="/img/docs/dbt-cloud/cost-management/filter-by-environment.png" width="70%" title="Filter the Discover view by environment."/>
- リストビューでは、本番環境が「PROD」アイコンで表示されます。
- 環境名の横にある色付きの四角をクリックすると、棒グラフビューに環境を追加または削除できます。
- 棒グラフにマウスポインターを合わせると、各環境のコストの内訳が表示されます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/environment-cost-breakdown.png" width="60%" title="The bar graph breaks down costs by environment."/>
- リストビューは、利用可能なフィールドで並べ替えることができます。リストビュー内の項目をクリックすると、コスト、クエリ実行回数、消費回数の詳細な棒グラフが表示されます。
    <Lightbox src="/img/docs/dbt-cloud/cost-management/individual-environment.png" width="60%" title="View individual environments and how they impact your costs."/>

## リソースの詳細

**Discover** タブからモデルまたはその他のリソースをクリックすると、モデルの消費量と実行に関する詳細情報が表示されます。

- 最初のセクションには、リソースの説明と情報（過去 30 日間のサイズと消費量クエリなど）が表示されます。
- **Lineage:** データ消費メトリクスを含む、さまざまなレンズの DAG ビューが表示されます。
- **Resource type:** 他のモデル、スナップショット、シード、メトリクスなど、モデルのリソースタイプの系統が表示されます。
- **Execution cost:** モデル系統内の各リソースの実行コストが表示されます。
- **Consuming query history:** モデル系統内のさまざまなリソースのウェアハウス消費メトリクスが表示されます。
- **Cost:** 過去 30 日間のリソース消費コストのグラフが表示されます。
- **Query execute count:** 過去 30 日間にリソースがウェアハウスに対して実行したクエリの合計数のグラフが表示されます。
- **消費クエリ:** ウェアハウスから分析データを収集するために使用されるクエリのグラフ。

<DocCarousel slidesPerView={1}>
<Lightbox src="/img/docs/dbt-cloud/cost-management/resource-description.png" width="90%" title="The resource description" />
<Lightbox src="/img/docs/dbt-cloud/cost-management/lens-resource-type.png" width="90%" title="The resource type lens" />
<Lightbox src="/img/docs/dbt-cloud/cost-management/lens-execution-cost.png" width="90%" title="The execution cost lens" />
<Lightbox src="/img/docs/dbt-cloud/cost-management/lens-consumption-query-history.png" width="90%" title="The consumption query history lens" />
<Lightbox src="/img/docs/dbt-cloud/cost-management/cost.png" width="90%" title="The cost analysis graph" />
<Lightbox src="/img/docs/dbt-cloud/cost-management/query-execution-count.png" width="90%" title="The query execution count history" />
<Lightbox src="/img/docs/dbt-cloud/cost-management/consumption-queries.png" width="90%" title="The consumption query history" />
</DocCarousel>

## 既知の制限事項

コスト管理ダッシュボードに関する既知の制限事項と注意事項を以下に示します。

- 現在、ダッシュボードには開発環境のコストは反映されません。
- ダッシュボードとデータプラットフォーム UI のコスト比較には、選択した期間または範囲によって異なる数値が反映される場合があり、差異が生じる可能性があります。
- コスト指標は、実行時間が非常に短いクエリを完全に反映しない可能性があり、平均値に歪みが生じる可能性があります。
- 消費指標には、分析ユースケースだけでなく、ウェアハウス内の特定モデルのすべてのクエリが含まれるため、リソース間の相対的な比較に最適です。
- 消費量メトリックは、dbt モデルをウェアハウス内のテーブルにマッピングすることに依存しているため、マッピングの変更方法によっては不正確になる可能性があります。
- dbt を実行すると、クエリが発行される実行ステップが複数回発生するため、直感的に判断しにくくなります。そのため、将来的には、より実行中心のメトリック（グループ化/集計）に移行していく予定です。
- コアコストは、dbt v1.10 以降を使用してクエリを dbt ワークロードに関連付けることに依存します。
- Snowflake が正確なコストデータを報告するまでに最大 72 時間かかるため、データが更新されるまでは過去 3 日間のコストが実際よりも少なくカウントされる可能性があります。