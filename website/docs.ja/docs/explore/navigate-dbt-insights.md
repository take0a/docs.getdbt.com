---
title: "Navigate the dbt Insights interface"
description: "Learn how to navigate the dbt Insights interface"
sidebar_label: "Navigation interface"
tags: [dbt Insights]
image: /img/docs/dbt-insights/insights-results.jpg
---

# dbt Insightsインターフェースを操作する <Lifecycle status="preview,managed,managed_plus" />

<IntroText>
<Constant name="query_page" /> インターフェースをナビゲートし、主要なコンポーネントを使用する方法を学習します。
</IntroText>

:::tip
<Constant name="query_page" /> は、Enterprise アカウント向けにプライベートベータ版としてご利用いただけます。ご参加いただくには、担当のアカウントマネージャーまでお問い合わせください。
:::

<Constant name="query_page" /> は、SQLクエリの作成、実行、分析のための対話型インターフェースを提供します。このセクションでは、<Constant name="query_page" /> の主なコンポーネントについて説明します。

## クエリコンソール
クエリコンソールは <Constant name="query_page" /> のメインコンポーネントです。SQL クエリの作成、実行、分析が可能です。クエリコンソールは以下の機能をサポートしています。
- クエリコンソールエディター。SQL クエリの作成、実行、分析が可能です。
  - 構文のハイライト表示とオートコンプリート機能をサポートしています。
  - SQL コード `ref` から対応するエクスプローラーページへのハイパーリンク
- [クエリコンソールメニュー](#query-console-menu)。**ブックマーク（アイコン）**、**開発**、**実行** ボタンが含まれています。
- [クエリ出力パネル](#query-output-panel)。クエリエディターの下にあり、クエリの結果を表示します。
  - **結果**、**詳細**、**チャート** の 3 つのタブがあり、クエリ実行を分析し、結果を視覚化できます。
- [クエリ コンソール サイドバー メニュー](#query-console-sidebar-menu)。**<Constant name="explorer" />**、**ブックマーク**、**クエリ履歴**、および **<Constant name="copilot" />** のアイコンが含まれています。

<Lightbox src="/img/docs/dbt-insights/insights-main.png" title="dbt Insights main interface with blank query editor" />

### クエリコンソールメニュー
クエリコンソールメニューは、クエリエディターの右上にあります。**ブックマーク**、**開発**、**実行** ボタンがあります。

- **ブックマーク** ボタン - よく使用する SQL クエリをお気に入りとして保存しておくと、簡単にアクセスできます。
  - **ブックマーク** をクリックすると、**ブックマーククエリの詳細** モーダル（ポップアップボックス）が表示され、**タイトル** と **説明** を入力できます。
  - [<Constant name="copilot" />](/docs/cloud/dbt-copilot) が自動でブックマークを作成します - AI アシスタントがブックマークのわかりやすい説明を自動的に生成します。
  - 新しく作成したブックマークには、[クエリコンソールサイドバーメニュー](#query-console-sidebar-menu) の **ブックマーク** アイコンからアクセスできます。
- **開発**: [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) または [<Constant name="visual_editor" />](/docs/cloud/canvas) を開いて、SQL クエリの編集を続行します。
- **実行** ボタン &mdash; SQL クエリを実行し、[結果] タブに結果を表示します。

  <Lightbox src="/img/docs/dbt-insights/develop-menu.png" title="dbt Insights Develop menu." />

## クエリ出力パネル

クエリ出力パネルはクエリエディターの下にあり、クエリの結果を表示します。以下のタブが表示され、クエリ実行を分析し、結果を視覚化できます。
- **結果** タブ - SQL 結果をページ区切りでプレビューします。
- **詳細** タブ - 実行された SQL クエリの簡潔な詳細を生成します。
  - クエリメタデータ - <Constant name="copilot" /> の AI 生成タイトルと説明。提供された SQL とコンパイル済み SQL も表示されます。
  - 接続の詳細 - 関連するデータプラットフォーム接続情報。
  - クエリの詳細 - クエリの実行時間、ステータス、列数、行数。
- **グラフ** タブ - 組み込みのグラフを使用して、クエリ結果を視覚化します。
  - グラフアイコンを使用して、結果を視覚化するグラフの種類を選択します。使用可能なグラフの種類は、**折れ線グラフ、棒グラフ、散布図** です。
  - **グラフ設定** を使用して、グラフの種類と視覚化する列をカスタマイズします。
  - 使用可能なグラフの種類は、**折れ線グラフ、棒グラフ、散布図** です。
- **ダウンロード** ボタン - 結果を CSV 形式でエクスポートできます

<DocCarousel slidesPerView={1}>
<Lightbox src="/img/docs/dbt-insights/insights-results.png" width="95%" title="dbt Insights Results tab" />
<Lightbox src="/img/docs/dbt-insights/insights-details.png" width="95%" title="dbt Insights Details tab" />
<Lightbox src="/img/docs/dbt-insights/insights-chart.png" width="95%" title="dbt Insights Chart tab" />
</DocCarousel>

## クエリコンソールのサイドバーメニュー
クエリコンソールのサイドバーメニューとアイコンには、次のオプションがあります。
- **<Constant name="explorer" /> アイコン** &mdash; 統合された <Constant name="explorer" /> ビューを使用して、プロジェクトのモデル、列、メトリックなどを表示します。
- **ブックマークアイコン** &mdash; よく使用するクエリを保存してアクセスします。
- **クエリ履歴アイコン** &mdash; 過去のクエリ、そのステータス（すべて、成功、エラー、保留中）、開始時刻、および所要時間を表示します。過去のクエリを検索し、ステータスでフィルタリングします。クエリ履歴からクエリを再実行することもできます。
- **<Constant name="copilot" /> アイコン** &mdash; [<Constant name="copilot" /> の AI アシスタント](/docs/cloud/dbt-copilot) を使用して、自然言語プロンプトを使用してクエリを変更または生成します。

<DocCarousel slidesPerView={1}>
<Lightbox src="/img/docs/dbt-insights/insights-explorer.png" width="90%" title="dbt Insights dbt Explorer icon" />
<Lightbox src="/img/docs/dbt-insights/insights-query-history.png" width="90%" title="dbt Insights Query history icon" />
<Lightbox src="/img/docs/dbt-insights/insights-copilot.png" width="60%" title="dbt Insights dbt Copilot" />
<Lightbox src="/img/docs/dbt-insights/manage-bookmarks.png" width="60%" title="Manage your query bookmarks" />
</DocCarousel>
