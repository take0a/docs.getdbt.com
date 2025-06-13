---
title: "カタログでデータを発見"
sidebar_label: "Discover data with Catalog"
description: "Learn about Catalog and how to interact with it to understand, improve, and leverage your dbt projects."
image: /img/docs/collaborate/dbt-explorer/example-project-lineage-graph.png
pagination_next: "docs/explore/data-health-signals"
pagination_prev: null
---

<IntroText>

<Constant name="explorer" /> を使用すると、プロジェクトの [リソース](/docs/build/projects) (モデル、テスト、メトリックなど)、それらの <Term id="data-lineage">系統</Term>、および [モデルの使用](/docs/explore/view-downstream-exposures) を表示して、最新の運用状態をより深く理解することができます。
</IntroText>

<Constant name="explorer" /> を使用して、<Constant name="cloud" /> 内のプロジェクトを移動および管理し、自分や他のデータ開発者、アナリスト、コンシューマーが dbt リソースを検出して活用できるようにします。<Constant name="explorer" /> は、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud)、[dbt <Constant name="query_page" />](/docs/explore/dbt-insights)、[<Constant name="orchestrator" />](/docs/deploy/deployments)、および [<Constant name="visual_editor" />](/docs/cloud/canvas) と統合され、dbt リソースの開発や表示に役立ちます。

## 前提条件

- [Starter、Enterprise、または Enterprise+ プラン](https://www.getdbt.com/pricing/) の <Constant name="cloud" /> アカウントが必要です。
- 探索するプロジェクトごとに、[本番環境](/docs/deploy/deploy-environments#set-as-production-environment) または [ステージング環境](/docs/deploy/deploy-environments#create-a-staging-environment) のデプロイメント環境がセットアップされている必要があります。
- デプロイメント環境で少なくとも 1 つのジョブが正常に実行されている必要があります。[CI ジョブ](/docs/deploy/ci-jobs) は <Constant name="explorer" /> を更新しないことに注意してください。
- <Constant name="explorer" /> ページが表示されています。これを行うには、<Constant name="cloud" /> のナビゲーションから [**探索**] を選択します。

import Generatemetadata from '/snippets.ja/_generate-metadata.md';

<Generatemetadata />

:::tip
組織でdbt CoreとCloudの両方を使用している場合は、dbt Coreのアーティファクトをdbt Cloudに自動的にアップロードし、<Constant name="explorer" />で表示することで、これらのワークフローを統合し、より連携したdbtエクスペリエンスを実現できます。詳細については、[ハイブリッドプロジェクト](/docs/deploy/hybrid-projects)をご覧ください。
:::

### 外部メタデータの取り込み <Lifecycle status="preview" />

[外部メタデータ取り込み](/docs/explore/external-metadata-ingestion)を使用してデータウェアハウスに直接接続することで、<Constant name="explorer" /> を使用して dbt で定義されていないテーブル、ビュー、その他のリソースを可視化できます。

dbt メタデータを作成し、外部メタデータを取得します。<Constant name="explorer" /> は、[Discovery API](/docs/dbt-cloud-apis/discovery-api) によって提供されるメタデータを使用して、プロジェクトの状態に関する詳細を表示します。利用可能なメタデータは、dbt プロジェクトで本番環境またはステージング環境として指定した [デプロイメント環境](/docs/deploy/deploy-environments) によって異なります。

## カタログの概要

:::info [グローバルナビゲーション](/docs/explore/explore-projects#search-resources) <Lifecycle status='self_service,managed,managed_plus' /> <Lifecycle status="preview" />

<Constant name="explorer" /> を使用すると、アカウント全体の dbt リソース（モデル、シード、スナップショット、ソース、エクスポージャーなど）を検索することで、検索範囲を拡張できます。これにより、返される結果の範囲が広がり、dbt プロジェクト全体のすべてのアセットについて、より詳細な情報を得ることができます。

グローバルナビゲーションを有効にするには:

- [所有者](/docs/cloud/manage-access/about-user-access#role-based-access-control) 権限を持つ開発者ライセンスが必要です。
- <Constant name="cloud" /> アカウントの [アカウント設定](/docs/cloud/account-settings) に移動し、[**dbt カタログのグローバルナビゲーションを有効にする**] チェックボックスをオンにします。

:::

<Constant name="explorer" /> 概要ページに移動して、プロジェクトのリソースとメタデータにアクセスします。このページには以下のセクションがあります。

- **検索バー** &mdash; キーワードでプロジェクト内のリソースを[検索](#search-resources)します。フィルターを使用して検索結果を絞り込むこともできます。
- **サイドバー** &mdash; 左側のサイドバーを使用して、**プロジェクトの詳細** セクションのモデル[パフォーマンス](/docs/explore/model-performance)と[プロジェクトの推奨事項](/docs/explore/project-recommendations)にアクセスできます。サイドバーの下部で、プロジェクトの[リソース、ファイルツリー、データベース](#browse-with-the-sidebar)を参照できます。
- プロジェクトの推奨事項は、プロジェクトのランディングページ内にあります。*
- **リネージグラフ** &mdash; プロジェクトまたはアカウントの[リネージグラフ](#project-lineage)を調べて、リソース間の関係を視覚化します。
- **最新の更新** - プロジェクトのリソースに関連する最新の変更や問題（最新のジョブ実行、変更されたプロパティ、リネージ、問題など）を表示します。
- **マートとパブリックモデル** - プロジェクトの [マート](/best-practices/how-we-structure/1-guide-overview#guide-structure-overview) と [パブリックモデル](/docs/mesh/govern/model-access#access-modifiers) を表示します。このビューから、アカウント内のすべてのパブリックモデルに移動することもできます。
- **モデルクエリ履歴** - [モデルクエリ履歴](/docs/explore/model-query-history) を使用して、モデルの使用状況クエリを追跡し、より深い分析情報を得ることができます。
- **ダウンストリームのエクスポージャーを視覚化** - Tableau から関連するデータモデルを自動的に公開して可視性を高めるには、[設定](/docs/cloud-integrations/downstream-exposures-tableau)と[ダウンストリームのエクスポージャーを視覚化](/docs/explore/view-downstream-exposures)を行います。
- **データヘルスシグナル** - 各リソースの [データヘルスシグナル](/docs/explore/data-health-signals) を表示して、その健全性とパフォーマンスを把握します。

### カタログ権限

グローバルナビゲーションを使用してプロジェクト全体を検索する場合、以下の権限が適用されます。

- プロジェクトのアクセス権限によって、グローバルナビゲーションの左側のメニューに表示される dbt プロジェクトが決まります。
- <Constant name="explorer" /> 検索ではソフトアクセス制御が使用され、検索結果には一致するすべてのリソースが表示されます。アクセス権のない項目には明確なインジケーターが表示されます。
- 外部メタデータの場合、グローバルプラットフォーム認証情報によって、メタデータユーザーが検出できるリソースが制御されます。詳細については、[外部メタデータの取り込み](/docs/explore/external-metadata-ingestion) をご覧ください。

import ExplorerCourse from '/snippets.ja/_explorer-course-link.md';

<ExplorerCourse />

## プロジェクトの系統グラフを調べる {#project-lineage}

<Constant name="explorer" /> は、プロジェクトの <Term id="dag">DAG</Term> を視覚的に表示し、操作できるようにします。プロジェクトの完全な系統グラフにアクセスするには、左側のサイドバーで [**概要**] を選択し、ページのメイン（中央）セクションにある [**系統グラフの参照**] ボタンをクリックします。

プロジェクトの系統グラフがすぐに表示されない場合は、[**系統グラフのレンダリング**] をクリックしてください。プロジェクトのサイズとコンピュータの使用可能なメモリによっては、グラフのレンダリングに時間がかかる場合があります。非常に大きなプロジェクトのグラフはレンダリングされない場合があるため、代わりにセレクタを使用してノードのサブセットを選択できます。

系統グラフ内のノードはプロジェクトのリソースを表し、エッジはノード間の関係を表します。ノードはリソースの種類に応じて色分けされ、アイコンが表示されます。

デフォルトでは、<Constant name="explorer" /> はプロジェクトの [適用状態](/docs/dbt-cloud-apis/project-state#definition-logical-vs-applied-state-of-dbt-nodes) 系統を表示します。つまり、プロジェクトで定義されているモデルだけでなく、正常にビルドされ、クエリに使用できるモデルも表示されます。

テストとマクロの系統グラフを調べるには、[リソースの詳細ページ](#view-resource-details) を参照してください。デフォルトでは、<Constant name="explorer" /> は、検索クエリの結果として返されない限り、これらのリソースを完全な系統グラフから除外します。

<Expandable alt_header="完全な系統グラフとどのように対話できますか?">

- グラフ内の任意の項目にマウスオーバーすると、リソースの名前とタイプが表示されます。
- マウススクロールでグラフを拡大/縮小できます。
- グラフとノードを掴んで移動できます。
- ノードを右クリック（コンテキストメニュー）すると、次の操作を実行できます。
    - ノード（上流ノードと下流ノードを含む）に再度フォーカスします。
    - ノードとその下流ノードのみに再度フォーカスします。
    - ノードとその上流ノードのみに再度フォーカスします。
    - ノードの [リソースの詳細](#view-resource-details) ページを表示します。
- リソースを選択すると、プロジェクト内の他のリソースとの関係が強調表示されます。グラフの右側にパネルが開き、リソースの詳細の概要が表示されます。サイドパネルには、説明、マテリアライズドタイプ、その他の詳細情報を表示する [全般] タブがあります。サイドパネルの右上隅には、次の操作を実行できます。
    - [リソースの詳細を表示](#view-resource-details) するには、[リソースの表示](#view-resource-details) アイコンをクリックします。
    - [IDE で開く](#open-in-ide) アイコンをクリックし、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用してリソースを調べます。
    - [ページへのリンクをコピー] アイコンをクリックし、ページのリンクをクリップボードにコピーします。
- [セレクタ](/reference/node-selection/methods) (検索バー内) を使用して、特定のリソースまたは DAG のサブセットを選択します。これにより、関心のあるリソースに焦点を絞り込むことができます。状態の比較 (結果、ソースのステータス、状態) を必要とするセレクタを除き、すべてのセレクタを使用できます。また、`--exclude` フラグと `--select` フラグ (オプション) も使用できます。例:
    - `resource_type:model [RESOURCE_NAME]` &mdash; 名前検索に一致するすべてのモデルを返します。
    - `resource_type:metric,tag:nightly` &mdash;タグ「nightly」が付いたメトリクスを返します。
- 検索バーの [グラフ演算子](/reference/node-selection/graph-operators) を使用して、特定のリソースまたは DAG のサブセットを選択します。これにより、関心のあるリソースに焦点を絞り込むことができます。例:
    - `+orders` &mdash; `orders` の上流ノードをすべて返します。
    - `+dim_customers,resource_type:source` &mdash; `dim_customers` の上流にあるすべてのソースを返します。
- 検索バーの [集合演算子](/reference/node-selection/set-operators) を使用して、特定のリソースまたは DAG のサブセットを選択します。これにより、関心のあるリソースに焦点を絞り込むことができます。例:
    - `+snowplow_sessions +fct_orders` &mdash; 結合演算には、スペースで区切られた引数を使用します。 `snowplow_sessions` または `fct_orders` のいずれかの上流ノードであるリソースを返します。

- グラフ内のノードを選択（ダブルクリック）して、[リソースの詳細を表示](#view-resource-details)します。
- <Constant name="explorer" /> [レンズ](#lenses)機能を使用するには、**レンズ**（グラフの右下隅）をクリックします。

</Expandable>

### 完全な系統グラフの例

プロジェクトの系統グラフでモデルを探索する例:

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-project-lineage-graph.png" width="100%" title="Example of full lineage graph" />

## レンズ

**レンズ**機能は、[プロジェクトの系統グラフ](#project-lineage) (右下隅) から利用できます。レンズはDAGのマップレイヤーのようなものです。レンズを使用すると、プロジェクトのコンテキストメタデータを大規模に把握しやすくなり、特に特定のモデルやモデルのサブセットを区別しやすくなります。

レンズを適用すると、系統グラフのノードにタグが表示され、レイヤーの値とその値に基づく色分けが示されます。大幅にズームアウトすると、グラフにはタグとその色のみが表示されます。

レンズは、DAGを拡大表示しているときにサブセットを分析したり、より広い視点からモデルや問題を見つけたりするのに役立ちます。

<Expandable alt_header="利用可能なレンズのリスト">

プロジェクト内のリソースは、リソースタイプ、マテリアライゼーションタイプ、モデルレイヤー、および最新の実行ステータスまたは最新のテストステータスによって特徴付けられます。レンズは、以下のメタデータで利用できます。

- **リソースタイプ**: モデル、テスト、シード、保存済みクエリ、[詳細](/docs/build/projects) など、リソースをリソースタイプ別に整理します。リソースタイプには、`resource_type` セレクタを使用します。
- **マテリアライゼーションタイプ**: データプラットフォームで dbt モデルを構築するための戦略を識別します。
- **最新のステータス**: 現在の環境でリソースを最後に実行したときのステータス。例: 障害が発生した DAG リージョンの診断。
- **モデルレイヤー**: [ベストプラクティスガイド](https://docs.getdbt.com/best-practices/how-we-structure/1-guide-overview#guide-structure-overview) に従って、モデルが属するモデリングレイヤー。例: 分析する marts モデルの検出。
    - **Marts** - プレフィックスが `fct_` または `dim_` のモデル、または `/marts/` サブディレクトリにあるモデル。
    - **Intermediate** - プレフィックスが `int_` のモデル。または `/int/` または `/intermediate/` サブディレクトリにあるモデル。
    - **Staging** - プレフィックスが `stg_` のモデル。または `/staging/` サブディレクトリにあるモデル。
- **テストステータス**: このリソースに対して実行されたテストの最新の実行時のステータス。モデルに結果が異なる複数のテストがある場合、レンズは「最悪のケース」のステータスを反映します。
- **消費クエリ履歴**: 特定の期間におけるこのリソースに対するクエリの数。

</Expandable>

### レンズの例

系統グラフを縮小表示した状態で**マテリアライゼーションタイプ** _レンズ_を適用した例です。このビューでは、各モデル名が、下部にあるマテリアライゼーションタイプの凡例に基づいて色分けされています。凡例にはマテリアライゼーションタイプが示されています。この色分けにより、異なるモデルのマテリアライゼーションタイプを素早く識別できます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-materialization-type.jpg" width="100%" title="Example of the Materialization type lens" />

**テスト ステータス** _lens_ を適用する例。各モデル名には、テスト ステータスを指定する下部の凡例に従ってテスト ステータスが表示されます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-test-status.jpg" width="100%" title="Example of the Test Status lens" />

## Keyword search {#search-resources}

<Constant name="explorer" /> を使用すると、グローバルナビゲーションによる検索機能が提供され、すべてのプロジェクトの dbt リソースだけでなく、Snowflake 内の dbt 以外のリソースも検索できます。

検索バーでキーワード検索を実行することで、プロジェクト内のリソースを見つけることができます。検索条件に一致するすべてのリソース名、列名、リソースの説明、データウェアハウスの関連、コードが、ページのメイン（中央）セクションにリストとして表示されます。正確な列名を検索すると、スキーマ内にその列を含むすべてのリレーショナルノードが結果に表示されます。一致するものがあった場合、検索結果に、リソースに指定された列が含まれていることを示す通知が表示されます。また、フィルターを適用して検索結果をさらに絞り込むこともできます。

<Expandable alt_header="検索機能">

- **部分キーワード検索** &mdash; あいまい検索とも呼ばれます。<Constant name="explorer" /> は「contains」ロジックを使用して検索結果を向上させます。つまり、検索語の正確なルートワードがわからなくても、部分的な語句を検索できます。
- **キーワードの除外** &mdash; 検索結果から除外するキーワードの先頭にマイナス記号 (-) を付けます。たとえば、「-user」と指定すると、そのキーワードに一致するすべての語句が検索結果から除外されます。
- **ブール演算子** &mdash; ブール演算子を使用してキーワード検索を拡張します。たとえば、「users OR github」の検索結果には、どちらかのキーワードに一致するものが含まれます。
- **フレーズ検索** &mdash; キーワードの文字列を二重引用符で囲むと、そのフレーズと完全に一致するものを検索できます (例: `"stg users"`)。詳細については、Wikipedia の [フレーズ検索](https://en.wikipedia.org/wiki/Phrase_search) をご覧ください。
- **SQL キーワード検索** &mdash; 検索には SQL キーワードを使用します。たとえば、「int github users connected」という検索結果には、特定のキーワード文字列を含む一致が含まれます（フレーズ検索と同様です）。

</Expandable>

<Expandable alt_header="フィルターサイドパネル">

キーワード検索を実行すると、**フィルター** サイドパネルが利用可能になります。このパネルを使用して、キーワード検索の結果を絞り込むことができます。デフォルトでは、<Constant name="explorer" /> はプロジェクト内のすべてのリソースを検索します。以下の項目でフィルターできます。

- [リソースタイプ](/docs/build/projects) (モデル、ソースなど)
- [モデルアクセス](/docs/mesh/govern/model-access) (パブリック、プライベートなど)
- [モデルレイヤー](/best-practices/how-we-structure/1-guide-overview) (マート、ステージングなど)
- [モデルの具体化](/docs/build/materializations) (ビュー、テーブルなど)
- [タグ](/reference/resource-configs/tags) (複数選択をサポート)

**モデル** オプションでは、モデルのプロパティ (アクセスタイプまたは具体化タイプ) でフィルターできます。また、**詳細** オプションも利用可能で、検索結果を列名、モデル コードなどに制限できます。

</Expandable>

<Expandable alt_header="グローバルナビゲーション">

<Constant name="explorer" /> は、従来のナビゲーション機能を基盤とし、ユーザーエクスペリエンスを向上させる画期的な新機能を導入しています。

- データアセットの検索 - アカウント全体の dbt リソース（モデル、シード、スナップショット、ソース、エクスポージャーなど）を検索することで、検索範囲を広げることができます。これにより、返される結果の幅が広がり、dbt プロジェクト全体のすべてのアセットについて、より深い洞察が得られます。
    - 外部メタデータの取り込み - データウェアハウスに直接接続することで、<Constant name="explorer" /> を使用して dbt で定義されていないテーブル、ビュー、その他のリソースを可視化できます。
- 系統の探索 - すべての dbt プロジェクト間のデータ関係を示すインタラクティブなマップを提供します。これにより、次のことが可能になります。
    - モデル、ソースなどの上流/下流の依存関係を表示します。
    - マルチプロジェクト（メッシュ）リンクを含む、プロジェクトレベルおよび列レベルの系統を詳細に分析します。
    - リソースタイプ、マテリアライゼーション、レイヤー、実行ステータスで「リネージレンズ」を使用してフィルタリングします。
    - 根本原因と下流への影響を追跡することで、データの問題をトラブルシューティングします。
    - DAG の低速部分、障害発生部分、または未使用部分を特定することで、パイプラインを最適化します。
- 推奨事項を確認 - プロジェクト全体の dbt の健全性のスナップショットを提供し、分析エンジニアリングを強化するための実用的なヒントを強調表示します。これらの分析情報は、<Constant name="cloud" /> メタデータとプロジェクト評価ルールセットのベストプラクティスを使用して自動的に生成されます。
- モデルクエリ履歴 - 各 dbt モデルがウェアハウスでクエリされる頻度を示し、次のことに役立ちます。
    - 成功した `SELECT` による実際の使用状況の追跡（ビルド/テストを除く）
    - 最適化または廃止のために、最も使用頻度が高い/最も使用頻度が低いモデルを特定します。
    - データドリブンな分析情報で投資とメンテナンスをガイドします。
- 下流のエクスポージャー - 接続されているすべてのプロジェクトにおいて、BI ツール、アプリ、ML モデル、レポートによって dbt モデルとソースがどのように使用されているかを示します。

</Expandable>

### キーワード検索の例
キーワード「customers」で検索し、モデル、説明、コードのフィルターを適用した結果の例です。[データヘルスシグナル](/docs/explore/data-health-signals)は、検索結果のモデル名の右側に表示されます。

## サイドバーで参照

サイドバーから、プロジェクトのリソース、ファイルツリー、データベースを参照できます。

- **リソース** タブ - プロジェクト内のすべてのリソースがタイプ別に整理されています。リストから任意のリソースタイプを選択すると、プロジェクト内のすべてのリソースがページのメインセクションにテーブルとして表示されます。モデル、メトリックなど、さまざまなリソースタイプの詳細については、[dbt プロジェクトについて](/docs/build/projects) を参照してください。
    - [データヘルスシグナル](/docs/explore/data-health-signals) は、リソース名の右側の [ヘルス] 列に表示されます。
- **ファイルツリー** タブ - プロジェクト内のすべてのリソースが、定義されているファイル別に整理されています。これは、dbt プロジェクトリポジトリ内のファイルツリーを反映しています。
- **データベース** タブ - プロジェクト内のすべてのリソースが、それらが構築されているデータベースとスキーマ別に整理されています。これは、プロジェクトの[適用された状態](/docs/dbt-cloud-apis/project-state)を表すデータ プラットフォームの構造を反映します。

## 統合ツールアクセス

[開発者ライセンス](/docs/cloud/manage-access/about-user-access#license-based-access-control)またはアナリストライセンスを持つユーザーは、<Constant name="cloud_ide" /> の <Constant name="explorer" /> から直接リソースを開いてモデルファイルを表示したり、<Constant name="query_page" /> でクエリを実行したり、<Constant name="visual_editor" /> でビジュアル編集したりできます。

## モデルのバージョンを表示

プロジェクト内のモデルがバージョン管理されている場合、モデルの詳細ページのタイトルとサイドバーのモデルリストで、適用されている [モデルのバージョン](/docs/mesh/govern/model-versions) （「プレリリース」、「最新」、「旧」）を確認できます。

## リソースの詳細を表示 {#view-resource-details}

プロジェクト内の任意のリソースの定義と最新の実行結果を表示できます。リソースを見つけて詳細を表示するには、系統グラフを操作したり、検索を使用したり、<Constant name="explorer" /> を参照したりできます。

表示される詳細（メタデータ）は、リソースの種類、定義、および本番環境のジョブ内で実行される [コマンド](/docs/deploy/job-commands) によって異なります。

リソースの詳細ページの右上隅では、次の操作を実行できます。
- [<Constant name="cloud_ide" /> で開く](#open-in-ide) アイコンをクリックすると、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用してリソースを確認できます。
- [共有] アイコンをクリックすると、ページのリンクをクリップボードにコピーできます。

<Expandable alt_header="モデルにはどのような詳細情報がありますか?">

- **データ ヘルス シグナル** &mdash; [データ ヘルス シグナル](/docs/explore/data-health-signals) を使用すると、データのヘルス状態を一目で確認できます。これらのアイコンは、モデルが「正常」、「注意」、「低下」、「不明」のいずれの状態であるかを示します。アイコンにマウス カーソルを合わせると、モデルのヘルス状態に関する詳細情報が表示されます。
- **ステータス バー** (ページ タイトルの下) &mdash; モデルの最終実行日時、実行の成否、データのマテリアライズ方法、行数、モデルのサイズに関する情報が表示されます。
- **全般** タブには次のものが含まれます。
    - **系統** グラフ &mdash; 操作可能なモデルの系統グラフ。グラフには、モデルの上流ノードと下流ノードが 1 つずつ表示されます。グラフの右上隅にある展開アイコンをクリックすると、モデルが完全な系統グラフ モードで表示されます。
    - **説明** セクション &mdash; [モデルの説明](/docs/build/documentation#プロジェクトに説明を追加する)。
    - **最近** セクション - モデルの最終実行日時、実行時間、実行の成否、ジョブ ID、実行 ID に関する情報。
    - **テスト** セクション - モデルの [テスト](/docs/build/data-tests)。最新のテスト ステータスを示すステータス インジケーターが含まれます。:white_check_mark: はテストが成功したことを示します。
    - **詳細** セクション - モデルのリレーション名（データ プラットフォームでどのように表現され、どのようにクエリできるか: `database.schema.identifier`）などの主要なプロパティ、アクセス、グループ、契約の有無などのモデル ガバナンス属性など。
    - **リレーションシップ** セクション -モデルが**依存**し、**参照**され、（該当する場合）モデルのプロジェクトを依存関係として宣言しているプロジェクトで**使用される**ノード。
- **コード** タブ - モデルのソースコードとコンパイル済みコード。
- **列** タブ - モデルで使用可能な列。このタブにはテスト結果（ある場合）も表示され、選択するとテストの詳細ページが表示されます。:white_check_mark: は合格したテストを示します。リソース内の列をフィルタリングするには、列ビューの上部にある検索バーを使用します。

</Expandable>

<Expandable alt_header="エクスポージャに関してどのような詳細情報を入手できますか?">

- **ステータスバー** (ページタイトルの下) - エクスポージャーが最後に更新された日時の情報。
- **データヘルスシグナル** - [データヘルスシグナル](/docs/explore/data-health-signals) を使用すると、データのヘルス状況を一目で確認できます。これらのアイコンは、リソースの状態が「正常」、「注意」、「劣化」のいずれであるかを示します。アイコンにマウスポインターを合わせると、エクスポージャーのヘルス状況に関する詳細情報が表示されます。
- **全般** タブには次のものが含まれます。
    - **データヘルス** - データの鮮度とデータ品質のステータス。
    - **ステータス** セクション - データの鮮度とデータ品質のステータス。
    - **リネージ** グラフ - エクスポージャーのリネージグラフ。グラフの右上隅にある [展開] アイコンをクリックすると、エクスポージャーを完全なリネージグラフ モードで表示できます。Tableau とネイティブに統合され、下流のリネージを自動生成します。
    - **説明** セクション - エクスポージャーの説明。
    - **詳細** セクション - エクスポージャーの種類、満期、所有者情報などの詳細。
    - **関係** セクション - エクスポージャーが **依存** するノード。

</Expandable>

<Expandable alt_header="テストではどのような詳細情報が得られますか?">

- **ステータス バー** (ページ タイトルの下) &mdash; テストの最終実行日時、テストの成功/不成功、テスト名、テスト ターゲット、列名に関する情報。指定がない場合は、すべてデフォルトで設定されます。
- **テスト タイプ** (ステータス バーの横) &mdash; 使用可能なテスト タイプ (単体テストまたはデータ テスト) に関する情報。指定がない場合は、すべてデフォルトで設定されます。

テストを選択すると、次の詳細が表示されます。
- **全般** タブには次の内容が含まれます。
    - **系統** グラフ &mdash; 操作可能なテストの系統グラフ。グラフには、テスト リソースからの上流ノードと下流ノードが 1 つずつ表示されます。グラフの右上隅にある展開アイコンをクリックすると、完全な系統グラフ モードでテストが表示されます。
    - **説明** セクション &mdash; テストの説明。
    - **最近** セクション &mdash;テストの最終実行日時、実行時間、テストの成功/不成功、ジョブID、実行IDに関する情報。
    - **詳細** セクション - スキーマ、重大度、パッケージなどの詳細。
    - **関係** セクション - テストが**依存**するノード。
    - **コード** タブ - テストのソースコードとコンパイル済みコード。

テストビューの例:

</Expandable>

<Expandable alt_header="ソース コレクション内の各ソース テーブルにはどのような詳細情報がありますか?">

- **ステータス バー** (ページ タイトルの下) - ソースが最後に更新された日時と、ソースが使用するテーブルの数に関する情報。
- **データ ヘルス シグナル** - [データ ヘルス シグナル](/docs/explore/data-health-signals) を使用すると、データのヘルス状態を一目で確認できます。これらのアイコンは、リソースの状態が「正常」、「注意」、「低下」のいずれであるかを示します。アイコンにマウス カーソルを合わせると、ソースのヘルス状態に関する詳細情報が表示されます。
- **全般** タブには次のものが含まれます。
    - **系統** グラフ - 操作可能なソースの系統グラフ。グラフには、ソースからの上流ノードと下流ノードが 1 つずつ表示されます。グラフの右上隅にある展開アイコンをクリックすると、完全な系統グラフ モードでソースが表示されます。
    - **説明** セクション - ソースの説明。
    - **ソースの鮮度** セクション -データの更新が成功したかどうか、ソースが最後に読み込まれた日時、実行によってデータが生成されたタイムスタンプ、実行 ID に関する情報。
    - **詳細** セクション - データベース、スキーマなどの詳細。
    - **リレーションシップ** セクション - 使用されたすべてのソースとその最新状態、最新状態が最後にチェックされた日時、ソースが最後に読み込まれた日時のタイムスタンプを一覧表示するテーブル。
- **列** タブ - ソースで使用可能な列。このタブには、テスト結果（ある場合）も表示され、選択するとテストの詳細ページが表示されます。:white_check_mark: は、テストが成功したことを示します。

</Expandable>

### Example of model details

<DocCarousel slidesPerView={1}>

Example of the details view for the model `customers`:<br /> <Lightbox src="/img/docs/collaborate/dbt-explorer/example-model-details.png" width="95%" title="Example of resource details" />

<Lightbox src="/img/docs/cloud-integrations/auto-exposures/explorer-lineage2.jpg" width="95%" title="Example of downstream exposure details for Tableau."/>

</DocCarousel>


## ステージング環境

<Constant name="explorer" /> は、本番環境に加えて、[ステージングデプロイメント環境](/docs/deploy/deploy-environments#staging-environment) のビューもサポートしています。これにより、本番環境と同じツールを使用しながら、本番前のデータワークフローを独自の視点で確認しながら、より詳細な調査が可能になります。

本番環境またはステージング環境のメタデータを調査し、データ開発ライフサイクルに役立てることができます。<Constant name="cloud" /> プロジェクトごとに [単一の環境](/docs/deploy/deploy-environments) を「production」または「staging」として設定し、適切なメタデータが生成されていることを確認すれば、<Constant name="explorer" /> でメタデータを表示できます。詳細については、[メタデータの生成](/docs/explore/explore-projects#generate-metadata) を参照してください。

## 関連コンテンツ
- [エンタープライズ権限](/docs/cloud/manage-access/enterprise-permissions)
- [モデルガバナンスについて](/docs/mesh/govern/about-model-governance)
- [データメッシュとは](https://www.getdbt.com/blog/what-is-data-mesh-the-definition-and-importance-of-data-mesh)に関するブログ
