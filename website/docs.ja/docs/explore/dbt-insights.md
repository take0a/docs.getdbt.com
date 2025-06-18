---
title: "About dbt Insights"
description: "Learn how to query data and perform exploratory data analysis using dbt Insights"
sidebar_label: "About dbt Insights"
tags: [Semantic Layer]
image: /img/docs/dbt-insights/insights-chart.jpg
---

# About dbt Insights <Lifecycle status="preview,managed,managed_plus" />

<IntroText>
<Constant name="query_page" /> を使用してデータをクエリする方法と、<Constant name="explorer" /> でドキュメントを表示する方法を学習します。
</IntroText>

:::tip
<Constant name="query_page" /> は、Enterprise アカウント向けにプライベートベータ版としてご利用いただけます。ご参加いただくには、担当のアカウントマネージャーまでお問い合わせください。
:::

<Constant name="cloud" /> の <Constant name="query_page" /> は、直感的でコンテキストリッチなインターフェースを通じて、ユーザーがシームレスにデータを探索およびクエリできるようにします。メタデータ、ドキュメント、AI支援ツール、そして強力なクエリ機能を1つの統合エクスペリエンスに統合することで、技術ユーザーとビジネスユーザーの橋渡しを実現します。

<Constant name="cloud" /> の <Constant name="query_page" /> は、[<Constant name="explorer" />](/docs/explore/explore-projects)、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud)、[<Constant name="visual_editor" />](/docs/cloud/canvas)、[<Constant name="copilot" />](/docs/cloud/dbt-copilot)、[<Constant name="semantic_layer" />](/docs/use-dbt-semantic-layer/dbt-sl) と統合され、探索的データ分析の実行、AI 支援ツールの活用、迅速な意思決定、チーム間のコラボレーションが容易になります。

<Lightbox src="/img/docs/dbt-insights/insights-main.gif" title="Overview of the dbt Insights and its features" />

## 主なメリット

主なメリットは次のとおりです。
- 構文のハイライト表示、タブ付きエディター、クエリ履歴などのツールを使用して、SQLクエリを迅速に作成、実行、反復処理できます。
- <Constant name="explorer" /> の dbt メタデータ、信頼シグナル、リネージを活用して、情報に基づいたクエリを構築できます。
- SQL、<Constant name="semantic_layer" /> クエリ、ビジュアルツールを活用して、さまざまな技術スキルレベルのユーザーがデータにアクセスできるようにします。
- <Constant name="copilot" /> の AI アシスタンスを使用して、SQLクエリや説明などを生成または編集できます。

ユースケースの例は次のとおりです。
- アナリストは、地域全体の販売パフォーマンス指標を分析し、結果を表示するためのクエリを迅速に作成できます。
- すべてのユーザーは、<Constant name="explorer" /> のエンドツーエンドの探索エクスペリエンスによって、充実した開発エクスペリエンスを享受できます。

## 前提条件

- <Constant name="cloud" /> [エンタープライズ層](https://www.getdbt.com/pricing) プランをご利用であること。<Constant name="query_page" /> の詳細については、[デモを予約](https://www.getdbt.com/contact) してください。
- すべての [テナント](/docs/cloud/about-cloud/tenancy) 構成で利用可能です。
- <Constant name="cloud" /> [開発者ライセンス](/docs/cloud/manage-access/seats-and-users) を保有し、<Constant name="query_page" /> にアクセスできること。
- [開発者認証情報](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud#get-started-with-the-cloud-ide) を設定済みであること。
- 本番環境と開発環境が <Constant name="cloud" /> の「最新」の [リリース トラック](/docs/dbt-versions/cloud-release-tracks) またはサポートされている dbt バージョンであること。
- サポートされているデータ プラットフォーム (Snowflake、BigQuery、Databricks、Redshift、または Postgres) を使用してください。
  - 開発ユーザー アカウントのシングル サインオン (SSO) はサポートされていますが、本番環境の資格情報の SSO はまだサポートされていません。
- (オプション) &mdash; <Constant name="query_page" /> から [<Constant name="semantic_layer" />](/docs/use-dbt-semantic-layer/dbt-sl) メトリクスをクエリするには、次の操作も行う必要があります。
  - dbt プロジェクトに対して <Constant name="semantic_layer" /> を [構成](/docs/use-dbt-semantic-layer/setup-sl) します。
  - <Constant name="semantic_layer" /> を構成した環境でジョブが正常に実行されるようにします。
