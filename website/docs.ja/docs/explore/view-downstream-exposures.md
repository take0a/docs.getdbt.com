---
title: "Visualize downstream exposures"
sidebar_label: "Visualize downstream exposures"
description: "Configure downstream exposures automatically from dashboards and understand how models are used in downstream tools for a richer downstream lineage."
pagination_prev: null
pagination_next:  "docs/explore/data-tile"
image: /img/docs/cloud-integrations/auto-exposures/explorer-lineage.jpg
---

# 下流のエクスポージャーを視覚化する <Lifecycle status="managed,managed_plus" />

<IntroText>
ダウンストリーム エクスポージャは Tableau とネイティブに統合され (Power BI は近日公開予定)、<Constant name="explorer" /> でダウンストリーム リネージを自動生成して、より豊富なエクスペリエンスを実現します。
</IntroText>

データチームにとって、データ製品の下流のユースケースとユーザーに関するコンテキストを把握することは非常に重要です。下流の[エクスポージャー](/docs/build/exposures)を自動的に活用することで、データチームは次のことが可能になります。

- 下流の分析におけるモデルの使用方法をより深く理解し、ガバナンスと意思決定を改善します。
- 上流のモデルを下流の依存関係にリンクすることで、インシデントを削減し、ワークフローを最適化します。
- サポートされているBIツールのエクスポージャー追跡を自動化し、リネージを常に最新の状態に保ちます。
- [エクスポージャーをオーケストレーション](/docs/cloud-integrations/orchestrate-exposures)により、スケジュールされたdbtジョブ中に基盤となるデータソースを更新し、タイムリーさを向上させ、コストを削減します。エクスポージャーのオーケストレーションは、基本的に[<Constant name="cloud" />ジョブスケジューラ](/docs/deploy/deployments)を使用してBIツールを定期的に更新する方法です。
  - エクスポージャーの視覚化とオーケストレーションの違いについて詳しくは、[ダウンストリーム エクスポージャーの視覚化とオーケストレーション](/docs/cloud-integrations/downstream-exposures) をご覧ください。

Tableau のダッシュボードからダウンストリーム エクスポージャーを自動的に構成する方法、前提条件などについては、[ダウンストリーム エクスポージャーの構成](/docs/cloud-integrations/downstream-exposures-tableau) をご覧ください。

### サポートされているプラ​​ン

ダウンストリームエクスポージャーは、すべての <Constant name="cloud" /> [エンタープライズプラン](https://www.getdbt.com/pricing/) でご利用いただけます。現在、同一サーバー上の単一の Tableau サイトのみに接続できます。

:::info Tableau Server
Tableau Server をご利用の場合は、<Constant name="cloud" /> リージョンの [<Constant name="cloud" /> の IP アドレスを許可リストに追加](/docs/cloud/about-cloud/access-regions-ip-addresses) する必要があります。
:::

import ViewExposures from '/snippets.ja/_auto-exposures-view.md';

<ViewExposures/>
