## ダウンストリーム エクスポージャーの表示

<Constant name="cloud" /> でダウンストリーム エクスポージャーを設定したら、[dbt Explorer](/docs/explore/explore-projects) で表示して、より詳細な情報を得ることができます。

ナビゲーションの [**Explore**] リンクをクリックして、dbt Explorer に移動します。[**Overview**] ページでは、以下のいくつかの場所でダウンストリーム エクスポージャーを表示できます。

<!-- no toc -->
- [Exposures menu](#exposures-menu)
- [File tree](#file-tree)
- [Project lineage](#project-lineage)

### Exposures menu
**リソース** の **エクスポージャー** メニュー項目から、ダウンストリームのエクスポージャーを表示できます。このメニューには、すべてのエクスポージャーの包括的なリストが表示されるため、すばやくアクセスして管理できます。メニューには以下の情報が表示されます。
- **名前**: エクスポージャーの名前。
- **ヘルス**: エクスポージャーの [データヘルスシグナル](/docs/explore/data-health-signals)。
- **タイプ**: エクスポージャーのタイプ (`dashboard` や `notebook` など)。
- **所有者**: エクスポージャーの所有者。
- **所有者のメール**: エクスポージャーの所有者のメールアドレス。
- **統合**: エクスポージャーが統合されている BI ツール。
- **エクスポージャーモード**: 定義されているエクスポージャーのタイプ (**自動** または **手動**)。
<Lightbox src="/img/docs/cloud-integrations/auto-exposures/explorer-view-resources.jpg" width="120%" title="View from the dbt Explorer under the 'Resources' menu."/>

### ファイルツリー
**ファイルツリー** 内の **imported_from_tableau** サブフォルダに直接アクセスできます。このビューでは、エクスポージャーがプロジェクトファイルにシームレスに統合されるため、プロジェクトの構造から簡単に見つけて参照できます。
<Lightbox src="/img/docs/cloud-integrations/auto-exposures/explorer-view-file-tree.jpg" width="120%" title="View from the dbt Explorer under the 'File tree' menu."/>
### Project lineage
**プロジェクト系統** ビューでは、プロジェクト内の依存関係と関係性が視覚的に表示されます。エクスポージャーは Tableau アイコンで表示され、プロジェクト全体のデータフローにどのように適合するかを直感的に確認できます。
<DocCarousel slidesPerView={1}>
<Lightbox src="/img/docs/cloud-integrations/auto-exposures/explorer-lineage2.jpg" width="95%" title="View from the dbt Explorer in your Project lineage view, displayed with the Tableau icon."/>
<Lightbox src="/img/docs/cloud-integrations/auto-exposures/explorer-lineage.jpg" width="95%" title="View from the dbt Explorer in your Project lineage view, displayed with the Tableau icon."/>
</DocCarousel>
