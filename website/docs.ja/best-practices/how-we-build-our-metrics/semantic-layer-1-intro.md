---
title: "dbt セマンティック レイヤーの概要"
description: Getting started with the dbt Semantic Layer
hoverSnippet: Learn how to get started with the dbt Semantic Layer
pagination_next: "best-practices/how-we-build-our-metrics/semantic-layer-2-setup"
pagination_prev: null
---

空飛ぶ車、ホバーボード、そして真のセルフサービス分析。これが私たちが約束された未来です。最初の 2 つはまだ数年先かもしれませんが、真のセルフサービス分析は今ここにあります。<Constant name="cloud" /> の<Constant name="semantic_layer" /> を使用すると、長年分析ツールを妨げてきた精度と柔軟性の間の緊張を解消し、組織内のすべての人が共通の指標の現実を探求できるようになります。分析エンジニアにとって最も良い点は、これらの新しいツールで構築することで、コードベースが大幅に [DRY](https://docs.getdbt.com/terms/dry) され、簡素化されることです。ご覧のとおり、dbt モデルと<Constant name="semantic_layer" />間の深い相互作用により、dbt プロジェクトは指標を作成するための理想的な場所になります。

## 学習目標

- ❓ **dbt <Constant name="semantic_layer" />** の **目的と機能**、特にそれを動かすエンジンとしての MetricFlow を理解します。
- 🧱 MetricFlow のコア コンポーネント (**セマンティック モデルとメトリック**) とそれらがどのように連携するかを理解していること。
- 🔁 <Constant name="semantic_layer" />の dbt モデルを **リファクタリング** する方法を理解します。
- 🏅 <Constant name="semantic_layer" />を最大限に活用するための**ベスト プラクティス** を認識します。

## ガイド構造の概要

1. dbt プロジェクトで **セットアップ** を実行します。
2. **セマンティック モデル** とその基本部分である **エンティティ、ディメンション、メジャー** を構築します。
3. **メトリック**を構築します。
4. **高度なメトリック** の定義: `ratio` および `derived` タイプ。
5. **ファイルとフォルダの構造**: 名前を付けるためのシステムを確立します。
6. <Constant name="semantic_layer" />のマートとロールアップを**リファクタリング**します。
7. **ベスト プラクティス** を確認します。

より少ないコードでより強力かつ柔軟な機能をユーザーに提供する準備ができたら、ぜひ始めましょう。

:::info
MetricFlow は、dbt でメトリックを定義するためのエンジンであり、[<Constant name="semantic_layer" />](/docs/use-dbt-semantic-layer/dbt-sl) の主要コンポーネントの 1 つです。SQL クエリの構築を処理し、dbt セマンティック モデルとメトリックの仕様を定義します。

外部統合を介して dbt メトリックをクエリする機能を含む、<Constant name="semantic_layer" />を完全に体験するには、[<Constant name="cloud" /> スターター、エンタープライズまたはエンタープライズ+ アカウント](https://www.getdbt.com/pricing/) が必要です。詳細については、[<Constant name="semantic_layer" /> FAQ](/docs/use-dbt-semantic-layer/sl-faqs) を参照してください。
:::
