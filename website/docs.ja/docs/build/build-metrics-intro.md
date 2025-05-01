---
title: "指標を構築する"
id: build-metrics-intro
description: "MetricFlow について学び、セマンティック モデルを使用してメトリックを構築します。"
sidebar_label: Build your metrics
tags: [Metrics, Semantic Layer, Governance]
hide_table_of_contents: true
pagination_next: "guides/sl-snowflake-qs"
pagination_prev: null
---

dbt で MetricFlow を使用して、メトリクスを一元的に定義します。
[dbt セマンティック レイヤー](/docs/use-dbt-semantic-layer/dbt-sl) の主要コンポーネントである MetricFlow は、SQL クエリの構築と、dbt セマンティック モデルおよびメトリクスの仕様定義を担当します。
セマンティック モデルやメトリクスなどの使い慣れた構成要素を使用することで、コーディングの重複を回避し、開発ワークフローを最適化し、企業メトリクスのデータガバナンスを確保し、データ コンシューマーの一貫性を保証します。

MetricFlow を使用すると、次のことが可能になります。
- dbt プロジェクトで直感的にメトリクスを定義
- [dbt Cloud CLI](/docs/cloud/cloud-cli-installation)、[dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud)、[dbt Core](/docs/core/installation-overview) など、お好みの環境で開発
- [MetricFlow コマンド](/docs/build/metricflow-commands) を使用して、開発環境でこれらのメトリクスをクエリおよびテスト
- ユニバーサル dbt セマンティック レイヤーの真の魔法を活用し、ダウンストリーム ツールでこれらのメトリクスを動的にクエリ (dbt Cloud [Team または Enterprise](https://www.getdbt.com/pricing/) アカウントのみで利用可能)

<div className="grid--3-col">

 <Card
    title="dbt クラウド セマンティック レイヤーのクイックスタート"
    body="このガイドを使用して、メトリックを構築および定義し、dbt セマンティック レイヤーを設定し、ダウンストリーム ツールを使用してクエリを実行します。"
    link="/guides/sl-snowflake-qs"
    icon="dbt-bit"/>

<Card
    title="MetricFlowについて"
    body="MetricFlow のコア概念、結合の使用方法、よく使用するクエリの保存方法、使用可能なコマンドについて理解します。"
    link="/docs/build/about-metricflow"
    icon="dbt-bit"/>

  <Card
    title="セマンティックモデル"
    body="データを定義する基盤としてセマンティックモデルを使用します。セマンティックモデルはセマンティックグラフのノードとして機能し、エンティティによって接続されます。"
    link="/docs/build/semantic-models"
    icon="dbt-bit"/>

  <Card
    title="メトリクス"
    body="メジャー、制約、または関数の強力な組み合わせを通じてメトリックを定義し、YAML ファイルまたは個別のファイルに簡単に整理します。"
    link="/docs/build/metrics-overview"
    icon="dbt-bit"/>
  
  <Card
    title="高度なトピック"
    body="データ モデリング ワークフローなど、dbt セマンティック レイヤーと MetricFlow の高度なトピックについて学習します。"
    link="/docs/build/advanced-topics"
    icon="dbt-bit"/>

  <Card
    title="dbtセマンティックレイヤーについて"
    body="データチームがメトリックを一元的に定義およびクエリできるようにするユニバーサルプロセスであるdbtセマンティックレイヤーのご紹介"
    link="/docs/use-dbt-semantic-layer/dbt-sl"
    icon="dbt-bit"/>

  <Card
    title="利用可能な統合"
    body="強力な dbt セマンティック レイヤーとシームレスに統合し、データ エコシステムから貴重な洞察をクエリして取得できるようにする多様なパートナーをご紹介します。"
    link="/docs/cloud-integrations/avail-sl-integrations"
    icon="dbt-bit"/>

</div> <br />

## 関連ドキュメント

- [dbt セマンティック レイヤー クイックスタート ガイド](/guides/sl-snowflake-qs)
- [dbt セマンティック レイヤー：今後の展望](https://www.getdbt.com/blog/dbt-semantic-layer-whats-next/) ブログ
- [dbt セマンティック レイヤー オンデマンド コース](https://learn.getdbt.com/courses/semantic-layer)
- [dbt セマンティック レイヤーに関する FAQ](/docs/use-dbt-semantic-layer/sl-faqs)
