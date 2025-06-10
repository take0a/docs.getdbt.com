---
title: "dbt セマンティックレイヤーを設定する"
description: Getting started with the dbt Semantic Layer
hoverSnippet: Learn how to get started with the dbt Semantic Layer
pagination_next: "best-practices/how-we-build-our-metrics/semantic-layer-3-build-semantic-models"
---

## はじめる

<Constant name="semantic_layer" />を含む dbt プロジェクトを開発するには、次の 2 つのオプションがあります。

- [<Constant name="cloud_cli" />](/docs/cloud/cloud-cli-installation) &mdash; MetricFlow コマンドは、<Constant name="cloud_cli" /> の `dbt sl` サブコマンドの下に埋め込まれています。これは、現時点では <Constant name="semantic_layer" /> コードを開発する最も簡単で機能豊富な方法です。任意のエディターを使用して、ターミナルからコマンドを実行できます。

- [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) &mdash; <Constant name="cloud_ide" /> でセマンティック モデルとメトリックを作成できます。

## 基本コマンド

- 🔍 <Constant name="semantic_layer" />で役立つ、あまり一般的ではないコマンドは `dbt parse` です。このコマンドはプロジェクトを解析し、**セマンティック マニフェスト** (プロジェクトで記述された意味のある接続の表現) を生成します。これは <Constant name="cloud" /> にアップロードされ、開発中に `dbt sl` コマンドを実行するために使用されます。このファイルは MetricFlow に **クエリを生成するための世界の状態** を提供します。
- 🧰 `dbt sl query` は、セマンティック レイヤーに対してクエリを実行し、結果のサンプルを返す、もう 1 つの優れたツールです。これは、セマンティック モデルとメトリックを構築する際にテストするのに最適です。たとえば、収益モデルを構築している場合は、`dbt sl query --metrics profit --group-by metric_time__month` を実行して、月間収益が正しく計算されていることを検証できます。
- 📝 最後に、`dbt sl list dimensions --metrics [metric name]` は、特定のメトリックで使用可能なすべてのディメンションを一覧表示します。これは、作業の進行に伴ってディメンションが増加しているかどうかを確認するのに役立ちます。<Constant name="semantic_layer" />の他の側面も `dbt sl list` できます。オプションの完全なリストを表示するには、`dbt sl list --help` を実行します。

使用可能なコマンドの詳細については、[MetricFlow コマンド](/docs/build/metricflow-commands) リファレンスを参照するか、コマンド ラインで `dbt sl --help` および `dbt sl [subcommand] --help` を使用してください。最初に dbt プロジェクトをセットアップする必要がある場合は、[クイックスタート ガイド](/docs/get-started-dbt) を確認してください。

## 前進！

ガイドの残りの部分では、架空のレストラン チェーンである Jaffle Shop プロジェクトに基づいたサンプル コードを紹介します。[Jaffle Shop リポジトリ](https://github.com/dbt-labs/jaffle-shop) でコードを自分で確認して試してみることができます。このガイドの後半で `food_revenue` などのメトリックを計算しているのを見たら、これが理由です。
