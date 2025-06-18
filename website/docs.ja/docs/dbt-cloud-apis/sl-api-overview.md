---
title: "Semantic Layer APIs"
id: sl-api-overview
description: "Integrate and query metrics and dimensions in downstream tools using the Semantic Layer APIs"
tags: [Semantic Layer, API]
hide_table_of_contents: true
pagination_next: "docs/dbt-cloud-apis/sl-jdbc"
---

# セマンティックレイヤーAPI <Lifecycle status="self_service,managed,managed_plus" />
 
現代のデータスタックにおける様々なツールの急速な増加は、データプロフェッショナルが様々なチームの多様なニーズに対応できるよう支援してきました。しかし、この増加のマイナス面は、チーム、ツール、ワークロード間でビジネスロジックが断片化していることです。<br /><br />

[<Constant name="semantic_layer" />](/docs/use-dbt-semantic-layer/dbt-sl) を使用すると、コード内でメトリクスを定義し（[MetricFlow](/docs/build/about-metricflow) を使用）、下流ツールでメトリクスやモデルなどの dbt で管理されるアセットに基づいてデータセットを動的に生成およびクエリできます。<Constant name="semantic_layer" /> との統合により、製品を使用する組織は、データに関してより効率的で信頼性の高い意思決定を行うことができます。また、重複コーディングの回避、開発ワークフローの最適化、データガバナンスの確保、データ利用者の一貫性の保証にも役立ちます。

<Constant name="semantic_layer" /> は、さまざまなツールやデータアプリケーションで使用できます。一般的なユースケースは以下のとおりです。

* ビジネスインテリジェンス (BI)、レポート作成、分析
* データ品質と監視
* ガバナンスとプライバシー
* データの検出とカタログ作成
* 機械学習とデータサイエンス

<!-- this partial lives here: https://github.com/dbt-labs/docs.getdbt.com/website/snippets/_sl-plan-info. Use it on diff pages and to tailor the message depending which instance can access the SL and what product lifecycle we're in. -->

<div className="grid--3-col">

<Card
    title="GraphQL API"
    body="GraphQL を使用して、下流ツールでメトリックとディメンションをクエリします。"
    link="/docs/dbt-cloud-apis/sl-graphql"
    icon="dbt-bit"/>

<Card
    title="JDBC API"
    body="JDBC ドライバーを使用して、下流ツールのメトリックとディメンションをクエリすると同時に、標準のメタデータ機能も提供します。"
    link="/docs/dbt-cloud-apis/sl-jdbc"
    icon="dbt-bit"/>

<Card
    title="Python SDK"
    body="Python SDK を使用して、Python で dbt セマンティック レイヤーと対話します。"
    link="/docs/dbt-cloud-apis/sl-python"
    icon="dbt-bit"/>

</div>
