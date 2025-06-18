---
title: "Data health signals"
sidebar_label: "Data health signals"
id: data-health-signals
description: "Learn how data health signals offer a quick, at-a-glance view of data health when browsing your resources in dbt Explorer."
image: /img/docs/collaborate/dbt-explorer/data-health-signal.jpg
---

# データヘルスシグナル <Lifecycle status="preview" />
データヘルスシグナルは、<Constant name="explorer" /> でリソースを参照する際に、データの健全性を一目で確認できるビューを提供します。**正常**、**注意**、**低下**、**不明** というインジケーターを使用して、リソースの健全性の状態を常に把握できます。

注: dbt 以外のリソースのデータヘルスは計算されません。

- サポートされているリソースは、[モデル](/docs/build/models)、[ソース](/docs/build/sources)、[エクスポージャー](/docs/build/exposures) です。
- 正確な健全性データを取得するには、リソースが最新であり、最近ジョブが実行されていることを確認してください。
- 各データヘルスシグナルは、テストの成功ステータス、リソースの説明の欠落、テストの欠落、30 日間の期間内にビルドがないなど、主要なデータヘルスコンポーネントを反映しています。[その他](#data-health-signal-criteria)。


<Lightbox src="/img/docs/collaborate/dbt-explorer/data-health-signal.jpg" width="55%" title="View data health signals for your models."/> 

## データヘルスシグナルへのアクセス

データヘルスシグナルには、以下の場所からアクセスできます。
- [検索機能](/docs/explore/explore-projects#search-resources)、または**リソース**タブの**モデル**、**ソース**、または**エクスポージャー**の下。
  - ソースの場合、データヘルスシグナルは[ソースの鮮度](/docs/deploy/source-freshness)ステータスも示します。
- [各リソースの詳細ページ](/docs/explore/explore-projects#view-resource-details)の**ヘルス**列。シグナルにマウスポインターを合わせるかクリックすると、詳細情報が表示されます。
- 公開モデルテーブルの**ヘルス**列。
- [DAG系統グラフ](/docs/explore/explore-projects#project-lineage)。任意のノードをクリックすると、ノードの詳細パネルが開き、ノードとその詳細が表示されます。
- [データ ヘルス タイル](/docs/explore/data-tile)では、埋め込み可能な iFrame を通じて BI ダッシュボードに表示されます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/data-health-signal.gif" width="95%" title="Access data health signals in multiple places in dbt Explorer."/> 

## データヘルスシグナルの基準

各リソースには、特定の基準セットによって決定されるヘルス状態があります。以下のタブを選択すると、そのリソースタイプの基準が表示されます。
<Tabs>
<TabItem value="models" label="Models">

モデルの健全性状態は、次の基準によって決定されます。
<!-- TODO: remove the 'tbd' lines in the table once meta 4025 is done -->
| **Health state** | **Criteria**   |
|-------------------|---------------|
| ✅ **Healthy**    | 以下のすべてが当てはまる必要があります:<br /><br />- 最後の実行で正常にビルドされている<br />- 過去 30 日以内にビルドされている<br />- モデルにテストが構成されている<br />- すべてのテストに合格している<br />- すべてのアップストリーム [ソースが最新](/docs/build/sources#source-data-freshness) であるか、鮮度が該当しない (`null` に設定されている)<br />- 説明がある |
| 🟡 **Caution**   | 次のいずれかに該当する必要があります: <br /><br />- 過去 30 日間にビルドされていない<br />- テストが構成されていない<br />- テストで警告が返される<br />- 1 つ以上のアップストリーム ソースが古くなっている:<br />&nbsp;&nbsp;&nbsp;&nbsp;- フレッシュネス チェックが構成されている<br />&nbsp;&nbsp;&nbsp;&nbsp;- 過去 30 日間にフレッシュネス チェックが実行された<br />&nbsp;&nbsp;&nbsp;&nbsp;- フレッシュネス チェックで警告が返された<br />- 説明がない |
| 🔴 **Degraded**  | 次のいずれかに該当する必要があります: <br /><br />- モデルのビルドに失敗しました<br />- モデルのテストに失敗しました<br />- 1 つ以上のアップストリーム ソースが古くなっています:<br />&nbsp;&nbsp;&nbsp;&nbsp;- 過去 30 日間にフレッシュネス チェックが実行されていません<br />&nbsp;&nbsp;&nbsp;&nbsp;- フレッシュネス チェックでエラーが返された |
| ⚪ **Unknown**    | - リソースの健全性を判断できません。リソースを処理するジョブ実行がありません。 |

</TabItem>

<TabItem value="sources" label="Sources">

The health state of a source is determined by the following criteria:

| **Health state** | **Criteria**   |
|-------------------|---------------|
| ✅ Healthy	| 以下のすべてが当てはまる必要があります: <br /><br />- フレッシュネス チェックが構成されている<br />- フレッシュネス チェックに合格している<br />- 過去 30 日間にフレッシュネス チェックが実行されている<br />- 説明がある |
| 🟡 Caution	| 次のいずれかに該当する必要があります: <br /><br />- フレッシュネス チェックで警告が返されました<br />- フレッシュネス チェックが構成されていません<br />- 過去 30 日間にフレッシュネス チェックが実行されていません<br />- 説明がありません |
| 🔴 Degraded	| - 鮮度チェックでエラーが返されました |
| ⚪ Unknown	| リソースの健全性を判断できません。リソースを処理するジョブ実行がありません。  |

</TabItem>

<TabItem value="exposures" label="Exposures">

露出の健康状態は、次の基準によって決定されます。

| **Health state** | **Criteria**   |
|-------------------|---------------|
| ✅ Healthy	| 以下のすべてが当てはまる必要があります: <br /><br />- 基礎となるソースが最新であること<br />- 基礎となるモデルが正常に構築されていること<br />- 基礎となるモデルのテストに合格していること<br /><!-- - 最新性は適用可能であること<br /> - (TBD) 基礎となるモデルが過去 30 日間に構築されていること --> |
| 🟡 Caution	| 次のいずれかが当てはまる必要があります: <br /><br />- 少なくとも 1 つの基礎ソースの鮮度チェックで警告が返されました<br />- 少なくとも 1 つの基礎モデルがスキップされました<br />- 少なくとも 1 つの基礎モデルのテストで警告が返されました<br /><!-- - (TBD) 少なくとも 1 つのモデルが過去 30 日間に構築されていません --> |   
| 🔴 Degraded	| 次のいずれかが当てはまる必要があります: <br /><br />- 少なくとも 1 つの基礎ソースの鮮度チェックでエラーが返されました<br />- 少なくとも 1 つの基礎モデルが正常にビルドされませんでした<br />- 少なくとも 1 つのモデルのテストでエラーが返されました |

</TabItem>

<!-- TODO: Add source collection health once META-3973/3971 are completed 
<TabItem value="source-collection" label="Source collection health">

The health state of a source collection is determined by the following criteria:

Functions as an aggregate of underlying sources

| **Health state** | **Criteria**   |
|-------------------|---------------|
| ✅ Healthy	| - All underlying sources have freshness checks configured OR<br />- All passed their freshness checks OR<br />- All freshness checks ran in the past 30 days OR<br /> - All sources have a description |
| 🟡 Caution	| - One or more sources lack freshness checks OR<br />- One or more freshness checks returned a warning OR<br />- One or more freshness checks not run in the past 30 days OR<br />- One or more sources missing a description |
| 🔴 Degraded	| - One or more underlying sources’ freshness checks returned error |

</TabItem>
-->

</Tabs>
