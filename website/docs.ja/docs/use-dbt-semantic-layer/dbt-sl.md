---
title: "dbt Semantic Layer"
id: dbt-sl
description: "dbt セマンティック レイヤーを使用して、データ チームがメトリックを一元的に定義およびクエリできるようにする方法について説明します。"
sidebar_label: "About the dbt Semantic Layer"
tags: [Semantic Layer]
hide_table_of_contents: false
pagination_next: "guides/sl-snowflake-qs"
pagination_prev: null
---

# dbt Semantic Layer <Lifecycle status="self_service,managed,managed_plus" />

<IntroText>

dbt セマンティック レイヤーは、データ チームが既存のモデルの上にメトリックを定義し、データ結合を自動的に処理できるようにすることで、重複したコーディングを排除します。

</IntroText>

[MetricFlow](/docs/build/about-metricflow) を搭載した dbt セマンティック レイヤーは、モデリング レイヤー（dbt プロジェクト）における「収益」などの重要なビジネス メトリクスの定義と使用のプロセスを簡素化します。メトリクス定義を一元管理することで、データ チームは下流のデータ ツールやアプリケーションからこれらのメトリクスへの一貫したセルフサービス アクセスを確保できます。

メトリクス定義を BI レイヤーからモデリング レイヤーに移動することで、データ チームは、使用するツールに関係なく、さまざまなビジネス ユニットが同じメトリクス定義に基づいて作業していることを確信できます。dbt でメトリクス定義が変更されると、呼び出されるすべての場所で更新され、すべてのアプリケーション間で一貫性が確保されます。安全なアクセス制御を確保するために、<Constant name="semantic_layer" />は堅牢な [アクセス許可](/docs/use-dbt-semantic-layer/setup-sl#set-up-dbt-semantic-layer) メカニズムを実装します。

詳細については、[<Constant name="semantic_layer" /> FAQ](/docs/use-dbt-semantic-layer/sl-faqs) または [ユニバーサル セマンティック レイヤーが必要な理由](https://www.getdbt.com/blog/universal-semantic-layer/) のブログ投稿を参照してください。

<div style={{ display: 'flex', justifyContent: 'center', }}>
<iframe width="850" height="510" position="relative" src="https://www.youtube.com/embed/DS7Ub_CmBR0?si=m92hLmxw1VuE6KKO" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

## dbtセマンティックレイヤーを使い始める

<!-- this partial lives here: https://github.com/dbt-labs/docs.getdbt.com/website/snippets/_sl-plan-info. Use it on diff pages and to tailor the message depending which instance can access the SL and what product lifecycle we're in. -->

import Features from '/snippets/_sl-plan-info.md'

<Features
product="dbt Semantic Layer"
plan="dbt Starter or Enterprise-tier"
/>

このページでは、<Constant name="semantic_layer" />の理解、構成、導入、統合に役立つさまざまなリソースを紹介します。以下のセクションには、各側面を詳しく説明した特定のページへのリンクが含まれています。<Constant name="semantic_layer" />を初めて設定する場合、メトリクスを導入する場合、または下流のツールと統合する場合など、これらのリンクを使用して必要な情報に直接移動できます。

<Constant name="semantic_layer" />の使用を開始するには、次のリソースを参照してください。
- [<Constant name="semantic_layer" />のクイックスタート](/guides/sl-snowflake-qs) - メトリクスを構築および定義し、<Constant name="semantic_layer" />を設定して、優れた統合機能を使用してクエリを実行します。
- [<Constant name="semantic_layer" /> FAQ](/docs/use-dbt-semantic-layer/sl-faqs) - 可用性、統合など、<Constant name="semantic_layer" />に関するよくある質問への回答をご覧ください。

## dbt セマンティック レイヤーを構成する

以下のリソースでは、<Constant name="semantic_layer" />を構成する方法について説明しています。
- [<Constant name="semantic_layer" />のセットアップ](/docs/use-dbt-semantic-layer/setup-sl) - 直感的なナビゲーションを使用して、<Constant name="cloud" /> で <Constant name="semantic_layer" />をセットアップする方法を学びます。
- [アーキテクチャ](/docs/use-dbt-semantic-layer/sl-architecture) - <Constant name="semantic_layer" />を構成する強力なコンポーネントについて詳しく説明します。

## メトリクスのデプロイ

このセクションでは、<Constant name="semantic_layer" />をデプロイし、メトリクスをマテリアライズする方法について説明します。
- [セマンティック レイヤーをデプロイする](/docs/use-dbt-semantic-layer/deploy-sl) - <Constant name="cloud" /> ジョブを実行して、<Constant name="semantic_layer" />をデプロイし、メトリクスをマテリアライズします。
- [エクスポートを使用してクエリを記述する](/docs/use-dbt-semantic-layer/exports) - エクスポートを使用して、よく使用するクエリをデータ プラットフォーム内でスケジュールに従って直接記述します。
- [よく使用するクエリをキャッシュする](/docs/use-dbt-semantic-layer/sl-cache) - よく使用するクエリに結果キャッシュと宣言型キャッシュを活用することで、パフォーマンスを向上させ、クエリの計算量を削減します。

## メトリクスの使用と統合

メトリクスを使用し、<Constant name="semantic_layer" />を下流のツールやアプリケーションと統合します。
- [メトリクスの使用](/docs/use-dbt-semantic-layer/consume-metrics) - <Constant name="semantic_layer" />を使用して、下流のツールやアプリケーションでメトリクスをクエリして使用します。
- [利用可能な統合](/docs/cloud-integrations/avail-sl-integrations) - <Constant name="semantic_layer" />と統合およびクエリできる幅広いパートナーを確認します。
- [<Constant name="semantic_layer" /> API](/docs/dbt-cloud-apis/sl-api-overview) - <Constant name="semantic_layer" /> API を使用して下流のツールでメトリクスをクエリし、一貫性と信頼性の高いデータメトリクスを実現します。

