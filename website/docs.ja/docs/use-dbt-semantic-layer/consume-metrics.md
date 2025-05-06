---
title: "セマンティックレイヤーからメトリクスを使用する"
description: "さまざまなツールと API を使用して、デプロイされた dbt セマンティック レイヤーからメトリックをクエリして使用する方法を学習します。"
sidebar_label: "Consume your metrics"
tags: [Semantic Layer]
pagination_next: "docs/use-dbt-semantic-layer/sl-faqs"
---

dbt セマンティックレイヤーを[デプロイ](/docs/use-dbt-semantic-layer/deploy-sl)したら、次に重要なのは、定義したメトリクスをクエリして使用することです。このページには、さまざまな[クエリ構文](/docs/dbt-cloud-apis/sl-jdbc#querying-the-api-for-metric-metadata)を使用して、さまざまな統合、API、ツール間でメトリクスを使用するプロセスをガイドする主要なリソースへのリンクがあります。

セマンティックレイヤーをデプロイしたら、さまざまなツールと API を使用してメトリクスをクエリできます。開始するための主なリソースは次のとおりです:

### 利用可能な統合

dbt セマンティックレイヤーを様々なビジネスインテリジェンス (BI) ツールやデータプラットフォームと統合することで、既存のワークフロー内でシームレスなメトリクスクエリが可能になります。以下の統合をご確認ください。

- [利用可能な統合](/docs/cloud-integrations/avail-sl-integrations) - Tableau、Google Sheets、Microsoft Excel など、dbt セマンティックレイヤーから直接メトリクスをクエリできる幅広いパートナーをご確認ください。

### API を使用したクエリ

dbt セマンティック レイヤーのパワーを最大限に活用するには、dbt セマンティック レイヤー API を使用してプログラムでメトリクスをクエリできます。
- [dbt セマンティック レイヤー API](/docs/dbt-cloud-apis/sl-api-overview)  &mdash;  dbt セマンティック レイヤー API を使用して下流ツールのメトリクスをクエリし、一貫性と信頼性の高いデータメトリクスを確保する方法を学習します。
  - [JDBC API クエリ構文](/docs/dbt-cloud-apis/sl-jdbc#querying-the-api-for-metric-metadata)  &mdash;  JDBC API を使用してメトリクスをクエリするための構文を、例と詳細な手順とともに学習します。
  - [GraphQL API クエリ構文](/docs/dbt-cloud-apis/sl-graphql#querying)  &mdash;  GraphQL API 経由でメトリクスをクエリするための構文を、例と詳細な手順とともに学習します。
  - [Python SDK](/docs/dbt-cloud-apis/sl-python#usage-examples) &mdash; Python SDK ライブラリを使用して、Python でプログラム的にメトリックをクエリします。
  
### 開発中のクエリ

dbt エコシステム内で作業する開発者にとって、開発フェーズ中に MetricFlow コマンドを使用してメトリクスをクエリする方法を理解することは不可欠です。
- [MetricFlow コマンド](/docs/build/metricflow-commands)  &mdash;  開発プロセス中に MetricFlow コマンドを使用してメトリクスを直接クエリする方法を学び、メトリクスが正しく定義され、期待どおりに動作していることを確認します。

## 次のステップ

メトリクスのクエリの基本を理解したら、設定を最適化し、メトリクス定義の整合性を確保することを検討してください。

- [クエリパフォーマンスの最適化](/docs/use-dbt-semantic-layer/sl-cache) &mdash; 宣言型キャッシュ技術を使用して、クエリの速度と効率を向上させます。
- [CI でのセマンティックノードの検証](/docs/deploy/ci-jobs#semantic-validations-in-ci) &mdash; 継続的インテグレーション (CI) ジョブでセマンティックノードを検証することで、dbt モデルへの変更によってメトリクスが破損しないことを確認します。
- [メトリクスとセマンティックモデルの構築](/docs/build/build-metrics-intro) &mdash; まだ行っていない場合は、お好みの開発ツールを使用してメトリクスとセマンティックモデルを定義および構築する方法を学習してください。
