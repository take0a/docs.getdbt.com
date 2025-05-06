---
title: "dbt Semantic Layer アーキテクチャ"
id: sl-architecture
description: "dbt セマンティック レイヤー製品のアーキテクチャと関連する質問。"
sidebar_label: "Semantic Layer architecture"
tags: [Semantic Layer]
---

dbtセマンティックレイヤーを使用すると、メトリクスを定義し、様々なインターフェースを使用してクエリを実行できます。セマンティックレイヤーは、クエリ対象のデータがデータプラットフォーム内のどこに存在するかを特定するという重労働を担い、リクエストを実行するためのSQL（結合の実行を含む）を生成します。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-architecture.jpg" width="85%" title="The diagram displays how your data flows using the dbt Semantic Layer and the variety of integration tools it supports."/>

## コンポーネント

dbt セマンティックレイヤーには、以下のコンポーネントが含まれます:


| Components | Information | dbt Core users | Developer plans |  Team plans | Enterprise plans | License |
| --- | --- | :---: | :---: | :---: | :---: | :---: |
| **[MetricFlow](/docs/build/about-metricflow)** | dbt の MetricFlow を使用すると、ユーザーは YAML 仕様を使用してセマンティック モデルとメトリックを一元的に定義できます。 | ✅ | ✅ | ✅ |  ✅  | BSL package (code is source available) |
| **dbt Semantic interfaces**| 指標、ディメンション、それらの相互リンク、そしてクエリの実行方法を定義するための設定仕様。[dbt-semantic-interfaces](https://github.com/dbt-labs/dbt-semantic-interfaces) はApache 2.0で利用可能です。 | ❌ | ❌ | ✅ | ✅ | Proprietary, Cloud (Team & Enterprise)|
| **Service layer** | クエリリクエストを調整し、関連するメトリッククエリをターゲットクエリエンジンにディスパッチします。これはdbt Cloudを通じて提供され、dbtバージョン1.6以降のすべてのユーザーが利用できます。サービスレイヤーには、データプラットフォームに対してSQLを実行するためのゲートウェイサービスが含まれています。 | ❌ | ❌ | ✅ | ✅ | Proprietary, Cloud (Team & Enterprise) |
| **[Semantic Layer APIs](/docs/dbt-cloud-apis/sl-api-overview)** | これらのインターフェースにより、ユーザーはGraphQLおよびJDBC APIを使用してメトリッククエリを送信できます。また、様々なツールとの高度な統合を構築するための基盤としても機能します。 | ❌ | ❌ | ✅ | ✅ | Proprietary, Cloud (Team & Enterprise)|


## 機能比較

以下の表は、dbt Cloud で利用可能な機能と dbt Core で利用可能な機能を比較したものです:

| Feature | MetricFlow Source available | dbt Semantic Layer with dbt Cloud |
| ----- | :------: | :------: |
| MetricFlow 仕様を使用して dbt でメトリックとセマンティック モデルを定義する | ✅ | ✅ |
| 設定ファイルのセットからSQLを生成する | ✅ | ✅ |
| コマンドラインインターフェース (CLI) を介してメトリックとディメンションをクエリする | ✅ | ✅ |
| CLI を介してディメンション、エンティティ、メトリックのメタデータをクエリする | ✅ | ✅ |
| セマンティック API (ADBC、GQL) を通じてメトリックとディメンションをクエリする  | ❌ | ✅ |
| ダウンストリーム統合 (Tableau、Hex、Mode、Google Sheets など) に接続します。 | ❌ | ✅ |
| エクスポートを作成して実行し、メトリック クエリをデータ プラットフォーム内のテーブルとして保存します。 | ❌ | ✅ |

## 関連ドキュメント
- [dbt セマンティックレイヤーに関するよくある質問](/docs/use-dbt-semantic-layer/sl-faqs)
