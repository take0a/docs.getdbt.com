---
title: "dbt クイックスタート"
id: get-started-dbt
hide_table_of_contents: true
pagination_next: null
pagination_prev: null
---

クイックスタートのいずれかを試して、dbt の旅を始めましょう。クイックスタートでは、[さまざまなデータ プラットフォーム](/docs/cloud/connect-data-platform/about-connections)を使用して [dbt Cloud](#dbt-cloud) または [dbt Core](#dbt-core) を設定するためのステップ バイ ステップ ガイドが提供されます。

## dbt Cloud

dbt Cloud は、単一の完全に管理されたソフトウェア サービスを使用してデータ製品を開発、テスト、展開、および調査できるスケーラブルなソリューションです。さまざまなスキルを持つチームが、あらゆる規模で信頼性の高いデータ製品を構築できるようにし、次のような機能を提供します。

- 複数のペルソナに合わせた開発エクスペリエンス (ブラウザ内の [dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) またはローカルの [dbt Cloud CLI](/docs/cloud/cloud-cli-installation))
- すぐに使用できる [CI/CD ワークフロー](/docs/deploy/ci-jobs)
- あらゆるエンドポイントに配信できる一貫したメトリックのための [dbt セマンティック レイヤー](/docs/use-dbt-semantic-layer/dbt-sl)
- マルチプロジェクトの [dbt Mesh](/best-practices/how-we-mesh/mesh-1-intro) セットアップによるデータのドメイン所有権
- 共同でデータの検出と理解を行う [dbt Explorer](/docs/collaborate/explore-projects)

[dbt Cloud](/docs/cloud/about-cloud/dbt-cloud-features) の詳細機能を確認し、[無料トライアルを開始](https://www.getdbt.com/signup/)して今すぐご利用ください。

<div className="grid--3-col">

<Card
    title="Quickstart for dbt Cloud and Amazon Athena"
    body="Integrate dbt Cloud with Amazon Athena for your data transformations."
    link="https://docs.getdbt.com/guides/athena"
    icon="athena"/>

<Card
    title="Quickstart for dbt Cloud and Azure Synapse Analytics"
    body="Discover how to integrate dbt Cloud with Azure Synapse Analytics for your data transformations."
    link="https://docs.getdbt.com/guides/azure-synapse-analytics"
    icon="azure-synapse-analytics"/>

<Card
    title="Quickstart for dbt Cloud and BigQuery"
    body="Discover how to leverage dbt Cloud with BigQuery to streamline your analytics workflows."
    link="https://docs.getdbt.com/guides/bigquery"
    icon="bigquery"/>

<Card
    title="Quickstart for dbt Cloud and Databricks"
    body="Learn how to integrate dbt Cloud with Databricks for efficient data processing and analysis."
    link="https://docs.getdbt.com/guides/databricks"
    icon="databricks"/>

<Card
    title="Quickstart for dbt Cloud and Microsoft Fabric"
    body="Explore the synergy between dbt Cloud and Microsoft Fabric to optimize your data transformations."
    link="https://docs.getdbt.com/guides/microsoft-fabric"
    icon="fabric"/>

<Card
    title="Quickstart for dbt Cloud and Redshift"
    body="Learn how to connect dbt Cloud to Redshift for more agile data transformations."
    link="https://docs.getdbt.com/guides/redshift"
    icon="redshift"/>

<Card
    title="Quickstart for dbt Cloud and Snowflake"
    body="Unlock the full potential of using dbt Cloud with Snowflake for your data transformations."
    link="https://docs.getdbt.com/guides/snowflake"
    icon="snowflake"/>

<Card
    title="Quickstart for dbt Cloud and Starburst Galaxy"
    body="Leverage dbt Cloud with Starburst Galaxy to enhance your data transformation workflows."
    link="https://docs.getdbt.com/guides/starburst-galaxy"
    icon="starburst"/>

<Card
    title="Quickstart for dbt Cloud and Teradata"
    body="Discover and use dbt Cloud with Teradata to enhance your data transformation workflows."
    link="https://docs.getdbt.com/guides/teradata"
    icon="teradata"/>

</div>

## dbt Core

[dbt Core](/docs/core/about-core-setup) は、データ実践者が分析エンジニアリングのベスト プラクティスを使用してデータを変換できるようにするコマンド ラインの [オープン ソース ツール](https://github.com/dbt-labs/dbt-core) です。手動のセットアップとカスタマイズを好む個人や小規模の技術チームに適しており、コミュニティ アダプターとオープン ソース標準をサポートしています。

<div className="grid--3-col">

<Card
    title="dbt Core from a manual install"
    body="Learn how to install dbt Core and set up a project."
    link="https://docs.getdbt.com/guides/manual-install"
    icon="dbt-bit"/>

<Card
    title="Quickstart for dbt Core using DuckDB"
    body="Learn how to connect to DuckDB."
    link="https://docs.getdbt.com/guides/duckdb?step=1"
    icon="duckdb"/>
</div>

## 関連ドキュメント

以下の追加リソースを利用して、dbt の知識と専門知識を広げてください:

- [毎月のデモに参加](https://www.getdbt.com/resources/webinars/dbt-cloud-demos-with-experts) して、dbt Cloud の動作を確認し、質問してください。
- [dbt Cloud AWS マーケットプレイス](https://aws.amazon.com/marketplace/pp/prodview-tjpcf42nbnhko) には、AWS に dbt Cloud をデプロイする方法、ユーザー レビューなどに関する情報が含まれています。
- [ベスト プラクティス](https://docs.getdbt.com/best-practices) には、構造、スタイル、セットアップに関する現在の視点から、dbt Labs がプロジェクトの構築にどのように取り組んでいるかに関する情報が含まれています。
- [dbt Learn](https://learn.getdbt.com) には、dbt の基礎、高度なトピックなどをカバーする無料のオンライン コースが用意されています。
- [dbt コミュニティに参加](https://www.getdbt.com/community/join-the-community)して、世界中の他のデータ実務者が dbt をどのように使用しているかを知り、自分の経験を共有し、dbt プロジェクトに関する支援を受けましょう。
