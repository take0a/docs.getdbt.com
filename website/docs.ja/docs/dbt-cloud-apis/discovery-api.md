---
title: "About the Discovery API"
pagination_next: "docs/dbt-cloud-apis/discovery-use-cases-and-examples"
---

# About the Discovery API <Lifecycle status="managed,managed_plus" />

<Constant name="cloud" /> はプロジェクトを実行するたびに、プロジェクトに関する情報を生成して保存します。メタデータには、プロジェクトのモデル、ソース、その他のノードに関する詳細情報と、それらの実行結果が含まれます。<Constant name="cloud" /> Discovery API を使用すると、この包括的な情報をクエリして、<Term id="dag">DAG</Term> とそれが生成するデータについてより深く理解することができます。

<Constant name="cloud" /> のメタデータを活用することで、データの監視とアラート、リネージの探索、自動レポート作成のためのシステムを構築できます。これにより、組織内のデータ検出、データ品質、パイプライン運用を改善できます。

Discovery API には、[アドホック クエリ](/docs/dbt-cloud-apis/discovery-querying)、カスタム アプリケーション、さまざまな [パートナー エコシステム統合](https://www.getdbt.com/product/integrations/) (BI/分析、カタログとガバナンス、品質と可観測性など)、および [モデル タイミング](/docs/deploy/run-visibility#model-timing) や [データ ヘルス タイル](/docs/explore/data-tile) などの <Constant name="cloud" /> 機能を使用してアクセスできます。

<Lightbox src="/img/docs/dbt-cloud/discovery-api/discovery-api-figure.png" width="80%" title="A rich ecosystem for integration "/>

<Constant name="cloud" /> メタデータに対してクエリを実行できます。

- [環境](/docs/environments-in-dbt) レベルでは、本番環境の <Constant name="cloud" /> プロジェクトの最新の状態（`environment` エンドポイントを使用）と過去の実行結果（`modelByEnvironment` を使用）の両方を取得できます。
- ジョブレベルでは、`models` や `test` などの特定のリソースタイプに対して実行された特定の <Constant name="cloud" /> ジョブの結果を取得できます。

<Snippet path="metadata-api-prerequisites" />

## Discovery API の用途

以下のタブをクリックすると、API のユースケース、実行可能な分析、そして API との統合によって得られる結果について詳しくご覧いただけます。

API を直接使用する場合、またはツールと統合する場合は、[ユースケースと例](/docs/dbt-cloud-apis/discovery-use-cases-and-examples) で詳細をご確認ください。

<Tabs>

<TabItem value="performance" label="Performance">

API を使用して、モデルビルド時間などの履歴情報を確認し、dbt プロジェクトの健全性を確認できます。オーケストレーション構成の非効率性を見つけることで、インフラストラクチャコストの削減とタイムリーさの向上につながります。その方法の詳細については、[パフォーマンス](/docs/dbt-cloud-apis/discovery-use-cases-and-examples#performance) をご覧ください。

例えば、[モデルタイミング](/docs/deploy/run-visibility#model-timing) タブを使用すると、モデルビルドのボトルネックを特定して最適化できます。

<Lightbox src="/img/docs/dbt-cloud/discovery-api/model-timing.png" width="200%" title="Model timing visualization in dbt"/>

</TabItem>

<TabItem value="quality" label="Quality">

API を使用して、テストの失敗、ソースの鮮度、実行ステータスをモニタリングすることで、データが正確かつ最新であるかどうかを判断できます。正確で信頼性の高い情報は、分析、意思決定、モニタリングに役立ち、組織が誤った意思決定を下すのを防ぎます。詳細については、[品質](/docs/dbt-cloud-apis/discovery-use-cases-and-examples#quality) をご覧ください。

[Webhook](/docs/deploy/webhooks) と併用すると、問題の検出、調査、アラート通知にも役立ちます。

</TabItem>

<TabItem value="discovery" label="Discovery">

API を使用すると、モデルやメトリックの定義、列情報などの情報を使用して、統合ツール内の dbt アセットを検索し、理解することができます。詳細については、[Discovery](/docs/dbt-cloud-apis/discovery-use-cases-and-examples#discovery) を参照してください。

データ生成者は関係者のためにデータを管理および整理する必要があり、データ利用者は大規模なデータを迅速かつ確実に分析し、情報に基づいた意思決定を行うことで、ビジネス成果の向上と組織のオーバーヘッドの削減を実現する必要があります。API は、カタログ、分析、アプリ、機械学習 (ML) ツールにおけるデータ検出エクスペリエンスに役立ちます。分析に使用するデータセットの起源と意味を理解するのに役立ちます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-model-details.png" width="200%" title="Data lineage produced by dbt" />

</TabItem>

<TabItem value="governance" label="Governance">

APIを使用して、モデルの開発者と使用者を確認し、ガバナンス向上のための標準的なプラクティスを確立しましょう。詳細については、[ガバナンス](/docs/dbt-cloud-apis/discovery-use-cases-and-examples#governance)をご覧ください。

</TabItem>

<TabItem value="development" label="Development">

APIを使用して、データセットの変更と使用状況を、エクスポージャー、リネージ、依存関係を調べることで確認できます。調査を通して、より効果的なdbtプロジェクトを定義および構築する方法を学ぶことができます。詳細については、[開発](/docs/dbt-cloud-apis/discovery-use-cases-and-examples#development)をご覧ください。



<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tile-pass.jpg" width="60%" title="Use exposures to embed data health tiles in your dashboards to distill trust signals for data consumers." />

</TabItem>


</Tabs>

## プロジェクト状態の種類

環境レベルで結果をクエリできる [プロジェクト状態](/docs/dbt-cloud-apis/project-state) には 2 種類あります。

- **定義** - プロジェクトが変更されると更新される、dbt プロジェクトの [リソース](/docs/build/projects) の論理状態。
- **適用済み** - dbt DAG 実行の成功時に出力され、データベースの状態を作成または記述します (例: `dbt run`、`dbt test`、ソースの鮮度など)。

これらの状態により、モデルの定義と適用済み状態の違いを簡単に調べることができ、「モデルは実行されたか？」「実行に失敗したか？」といった疑問に答えることができます。適用済みモデルは、最後に成功した実行に基づいて、データ プラットフォーム内のテーブル/ビューとして存在します。

## 関連ドキュメント

- [Discovery API のユースケースと例](/docs/dbt-cloud-apis/discovery-use-cases-and-examples)
- [Discovery API のクエリ](/docs/dbt-cloud-apis/discovery-querying)
- [スキーマ](/docs/dbt-cloud-apis/discovery-schema-job)
