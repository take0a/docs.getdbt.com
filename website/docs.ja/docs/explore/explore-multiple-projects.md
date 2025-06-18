---
title: "Explore multiple projects"
sidebar_label: "Explore multiple projects"
description: "Learn about project-level lineage in dbt Explorer and its uses."
---

アカウント内のすべてのプロジェクトとパブリック モデル (パブリック モデルが定義されている場所) を表示し、プロジェクト間のリソースとその使用方法をより深く理解します。

import ExplorerCourse from '/snippets.ja/_explorer-course-link.md';

<ExplorerCourse />

プロジェクトのリソースレベルの系統グラフには、DAG 内のプロジェクト間の関係が表示されます。**PRJ** アイコンは、それがプロジェクトリソースであるかどうかを示します。このアイコンはノード名の左側にあります。

プロジェクトレベルの系統グラフを表示するには、メインの概要ページの右上隅にある **系統の表示** アイコンをクリックします。
- このビューには、アカウント内のすべてのプロジェクトとその関係が表示されます。
- 上流（親）プロジェクトを表示すると、そのプロジェクトに依存する下流（子）プロジェクトが表示されます。
- モデルを選択すると、系統内の依存プロジェクトが表示されます。
- 上流（親）プロジェクトをクリックすると、**関係** タブにそのプロジェクトを参照している他のプロジェクトが表示され、それらに依存する下流（子）プロジェクトの数が表示されます。
  - これには、直接 `{{ ref() }}` が指定されていなくても、上流プロジェクトを `dependencies.yml` ファイルで依存関係としてリストしているすべてのプロジェクトが含まれます。
- パブリック モデルからプロジェクト ノードを選択すると、[権限](/docs/cloud/manage-access/enterprise-permissions) がある場合に詳細な系統グラフが開きます。

:::tip Indirect dependencies
プロジェクトの系統を表示すると、<Constant name="explorer" /> には [直接](/docs/mesh/govern/project-dependencies) 参照されているパブリックモデルのみが表示されます。[間接的な依存関係](/faqs/Project_ref/indirectly-reference-upstream-model) は表示されません。プロジェクト内の参照モデルが別の上流パブリックモデルに依存している場合、第 2 レベルのモデルは <Constant name="explorer" /> には表示されませんが、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) 系統ビューには表示されます。
:::

<Lightbox src="/img/docs/collaborate/dbt-explorer/cross-project-lineage-parent.png" width="100%" height="100" title="View your cross-project lineage in a parent project and the other projects that reference it by clicking the 'Relationships' tab."/>

上流（親）プロジェクトからパブリックモデルをインポートおよび参照している下流（子）プロジェクトを表示する場合：
- パブリックモデルは系統グラフに表示され、クリックするとモデルの詳細を表示できます。
- モデルをクリックすると、そのモデルを生成した特定の <Constant name="cloud" /> プロジェクト、説明、パッケージなど、モデルに関する一般情報を含むサイドパネルが開きます。
- 別のプロジェクトのモデルをダブルクリックすると、権限があれば、親プロジェクトのリソースレベルの系統グラフが開きます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/cross-project-child.png" width="100%" height="100" title="View a downstream (child) project that imports and refs public models from the upstream (parent) project."/>

## プロジェクトレベルの系統グラフを調べる

プロジェクト間のコラボレーションでは、[プロジェクトの系統グラフを調べる](/docs/explore/explore-projects#project-lineage)で説明されているのと同じ方法でDAGを操作できますが、プロジェクトレベルで操作して詳細を表示することもできます。

アカウント内のプロジェクトに対する権限を持っている場合は、アカウント全体で使用されているすべての公開モデルを表示できます。ただし、公開モデルの詳細と非公開モデルを表示するには、それらのモデルが定義されている特定のプロジェクトに対する権限が必要です。

アカウント内のすべてのプロジェクトを表示するには（系統グラフまたはリストビューで表示）：
- **Explore** ページの左上、ナビゲーションバーの近くに移動します。
- プロジェクト名にマウスを合わせ、アカウント名を選択します。アカウントレベルの系統グラフページに移動し、アカウント内のすべてのプロジェクト（異なるプロジェクト間の依存関係や関係を含む）を表示できます。
- ページの右上隅にある **リスト表示** アイコンをクリックすると、アカウント内のすべてのプロジェクトのリスト表示が表示されます。
- リスト表示ページには、公開モデルリスト、プロジェクトリスト、プロジェクト検索用の検索バーが表示されます。
- ページの右上隅にある **系統表示** アイコンをクリックすると、アカウントレベルの系統グラフが表示されます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/account-level-lineage.gif" width="100%" title="View a downstream (child) project, which imports and refs public models from upstream (parent) projects."/>

アカウントレベルの系統グラフでは、次の操作を実行できます。

- グラフの右上にある **系統ビュー** アイコンをクリックすると、プロジェクト間の系統グラフが表示されます。
- グラフの右上にある **リストビュー** アイコンをクリックすると、プロジェクトリストが表示されます。
  - [**プロジェクト**] タブからプロジェクトを選択すると、そのプロジェクトのメインの **エクスプローラ** ページに切り替わります。
  - [**公開モデル**] タブからモデルを選択すると、[モデルの詳細ページ](/docs/explore/explore-projects#view-resource-details) が表示されます。
  - 検索バーを使用してプロジェクトを検索できます。
- グラフ内のプロジェクトノードを選択（ダブルクリック）すると、そのプロジェクトの系統グラフに切り替わります。

グラフ内のプロジェクトノードを選択すると、グラフの右側にプロジェクトの詳細パネルが開き、次の操作を実行できます。

- プロジェクトで定義されているリソースの数を表示します。
- 公開モデルのリスト（存在する場合）を表示します。
- プロジェクトを使用している他のプロジェクトのリスト（存在する場合）を表示します。
- **「プロジェクト系統を開く」** をクリックして、プロジェクトの系統グラフに切り替えます。
- **「共有」** アイコンをクリックして、プロジェクトパネルのリンクをクリップボードにコピーし、グラフを他のユーザーと共有します。

<Lightbox src="/img/docs/collaborate/dbt-explorer/multi-project-overview.gif" width="95%" title="Select a downstream (child) project to open the project details panel for resource counts, public models associated, and more. "/>
