---
title: "Model performance"
sidebar_label: "Model performance"
description: "Learn about the performance of your models so you can make improvements to save time and money."
---

# Model performance <Lifecycle status="managed,managed_plus" />

<Constant name="explorer" /> は、<Constant name="cloud" /> 実行に関するメタデータを提供し、詳細なモデルパフォーマンスと品質分析を可能にします。この機能は、モデルのリファクタリングやジョブ設定の調整など、プロジェクトやデプロイメントの微調整が必​​要な箇所をハイライトすることで、インフラストラクチャコストの削減とデータチームの時間節約に役立ちます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/explorer-model-performance.gif" width="100%" title="Overview of Performance page navigation."/>

import ExplorerCourse from '/snippets/_explorer-course-link.md';

<ExplorerCourse />

## パフォーマンス概要ページ

パフォーマンス概要ページを使用すると、パフォーマンス改善が必要な領域を特定できます。このページでは、すべてのプロジェクトモデルを包括的に分析し、実行時間が最も長いモデル、最も頻繁に実行されるモデル、実行/テスト中に失敗率が最も高いモデルを表示します。データは環境とジョブタイプ別にセグメント化できるため、以下の点に関する洞察が得られます。

- 最も実行されたモデル（合計数）。
- 実行時間が最も長いモデル（平均所要時間）。
- 失敗が最も多いモデル。実行失敗（割合と件数）とテスト失敗（割合と件数）の詳細。

各データポイントは、<Constant name="explorer" /> 内の個々のモデルにリンクされています。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-performance-overview-page.png" width="90%" title="Example of Performance overview page"/>

過去3か月までのメタデータ履歴を表示できます。フィルターを使用して期間を選択してください。デフォルトでは2週間前まで遡って表示されます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/ex-2-week-default.png" width="55%" title="Example of dropdown"/>

## モデルパフォーマンスタブ

モデルパフォーマンスタブを使用して、過去のパフォーマンス分析を行うことで、実行時間、実行回数、失敗回数の傾向を確認できます。日次実行データには以下が含まれます。

- モデルの平均実行時間
- モデルの実行回数（失敗/エラーを含む、合計）

データポイントをクリックすると、その日のすべてのジョブ実行がリストされた表が表示されます。各行には、特定の実行の詳細への直接リンクがあります。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-model-performance-tab.png" title="Example of the Model performance tab"/> 

