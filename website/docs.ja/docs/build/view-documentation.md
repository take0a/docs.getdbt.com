---
title: "ドキュメントを見る"
description: "Learn how robust documentation for your dbt models helps stakeholders discover and understand your datasets."
id: "view-documentation"
---

dbt は、dbt ドキュメントを表示するための直感的でスケーラブルなツールを提供します。詳細なドキュメントは、開発者やその他の関係者が dbt プロジェクトのコンテキストを共有する上で不可欠です。

ニーズに応じて、2 つの補完的な方法でドキュメントを表示できます:

| Option | Description | Availability |
|------|-------------|--------------|
| [**dbt Docs**](#dbt-docs) | モデルの系統、メタデータ、および Web サーバー (S3 や Netlify など) でホストできるドキュメントを含む静的 Web サイトを生成します。 | dbt Core or dbt Cloud Developer plans |
| [**dbt Explorer**](/docs/collaborate/explore-projects) | dbt Cloud のプレミア ドキュメント エクスペリエンス。dbt Docs を基盤として、豊富な [メタデータ](/docs/collaborate/explore-projects#generate-metadata)、カスタマイズ可能なビュー、プロジェクトとリソースに関する詳細な情報、共同作業ツールを備えた動的なリアルタイム インターフェースを提供します。 | dbt Cloud Team or Enterprise plans |

## ドキュメントのナビゲーション
次のセクションでは、dbt Explorer および dbt Docs でドキュメントをナビゲートする方法について説明します。

### dbt Explorer <Lifecycle status="team,enterprise" />

[dbt Explorer](/docs/collaborate/explore-projects) は、モデル、ソース、系統を動的かつインタラクティブに探索する方法を提供します。
dbt Explorer にアクセスするには、dbt Cloud ナビゲーション メニューの **Explore** オプションに移動します。

<DocCarousel slidesPerView={1}>

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-model-details.png" width="95%" title="Example of dbt Explorer's resource details page and its lineage." />

<Lightbox src="/img/docs/collaborate/dbt-explorer/explorer-main-page.gif" width="95%" title="Navigate dbt Explorer to discover your project's resources and lineage."/>

</DocCarousel>

dbt Explorer は、データ プロジェクトのナビゲーションと理解を強化するための包括的な機能スイートをユーザーに提供します。たとえば、次のようになります。

- プロジェクトの DAG のインタラクティブな系統の視覚化により、リソース間の関係を理解できます。
- 包括的なフィルターを備えたリソース検索バーにより、プロジェクト リソースを効率的かつ迅速に見つけることができます。
- モデル パフォーマンスの分析情報により、dbt Cloud 実行のメタデータにアクセスして、モデルのパフォーマンスと品質を詳細に分析できます。
- データ エステート全体のテスト カバレッジとドキュメントを改善するための提案を含むプロジェクトの推奨事項。
- データ ヘルス シグナルにより、データ ヘルス インジケーターを介して各リソースのヘルスとパフォーマンスを監視できます。
- モデル クエリ履歴により、モデルに対する消費クエリを追跡して、データの使用状況に関するより深い分析情報を取得できます。
- ダウンストリーム エクスポージャーにより、Tableau などのツールから関連するデータ モデルを自動的に公開して、可視性を高めます。

系統の探索、リソースのナビゲート、モデル クエリ履歴とデータ ヘルス シグナルの表示、機能の可用性などに関する詳細と手順については、[dbt Explorer でデータを検出する](/docs/collaborate/explore-projects) を参照してください。

### dbt Docs

dbt Docs は、dbt Core または dbt Cloud Developer プラン プロジェクトに関する貴重な情報を提供します。このインターフェースを使用すると、特定のモデルのドキュメントに移動できます。次のような表示になります:

<Lightbox src="/img/docs/building-a-dbt-project/testing-and-documentation/f2221dc-Screen_Shot_2018-08-14_at_6.29.55_PM.png" title="Auto-generated documentation for a dbt model"/>

ここでは、プロジェクト構造の表現、モデルのマークダウン説明、モデル内のすべての列のリスト (ドキュメント付き) を確認できます。

dbt Docs ページから、Web ページの右下隅にある緑色のボタンをクリックして、DAG の「ミニマップ」を展開します。このペインには、探索しているモデルの直接の親と子が表示されます。

<Lightbox src="/img/docs/building-a-dbt-project/testing-and-documentation/ec77c45-Screen_Shot_2018-08-14_at_6.31.56_PM.png" title="Opening the DAG mini-map"/>

この例では、`fct_subscription_transactions` モデルには直接の親が 1 つだけあります。ウィンドウの右上隅にある [展開] ボタンをクリックすると、グラフを水平方向に回転して、モデルの完全な <Term id="data-lineage">系統</Term> を表示できます。この系統は、`--select` フラグと `--exclude` フラグを使用してフィルタリングできます。これらは、[モデル選択構文](/reference/node-selection/syntax) のセマンティクスと一致しています。さらに、右クリックして DAG を操作したり、ドキュメントにジャンプしたり、グラフの視覚化へのリンクを同僚と共有したりできます。

<Lightbox src="/img/docs/building-a-dbt-project/testing-and-documentation/ac97fba-Screen_Shot_2018-08-14_at_6.35.14_PM.png" title="The full lineage for a dbt model"/>

## ドキュメントサイトを展開する

dbt Explorer または dbt Docs でドキュメントを簡単に展開し、チームが利用できるようにします。

:::caution セキュリティ

`dbt docs serve` コマンドは、ドキュメント サイトのローカル/開発ホスティングのみを対象としています。ドキュメント サイトが安全にホストされていることを確認するには、次のセクションに記載されている方法のいずれか (または同様の方法) を使用してください。

:::

### dbt Explorer <Lifecycle status="team,enterprise" />


dbt Explorer は、生成されたメタデータを使用して、各本番ジョブまたはステージング ジョブの実行後にドキュメントを自動的に更新します。つまり、手動での展開が不要で、プロジェクトの最新の結果が常に得られます。dbt Explorer がメタデータを使用してドキュメントを自動的に更新する方法の詳細については、[メタデータの生成](/docs/collaborate/explore-projects#generate-metadata) を参照してください。

ドキュメント サイトを展開する方法については、[dbt Cloud でドキュメントを構築および表示する](/docs/collaborate/build-and-view-your-docs) を参照してください。

### dbt Docs
dbt Docs は、Web 上で簡単にホストできるように構築されています。このサイトは「静的」であるため、ドキュメントを提供するのに「動的」サーバーは必要ありません。ドキュメントをホストする方法はいくつかあります:

* [Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/WebsiteHosting.html) でホストする (オプションで [IP アクセス制限あり](https://docs.aws.amazon.com/AmazonS3/latest/dev/example-bucket-policies.html#example-bucket-policies-use-case-3))
* [Netlify](https://discourse.getdbt.com/t/publishing-dbt-docs-to-netlify/121) で公開する
* Apache/Nginx などの独自の Web サーバーを使用する
* dbt Cloud Developer プランをご利用の場合は、[dbt Cloud でドキュメントを構築および表示する](/docs/collaborate/build-and-view-your-docs#dbt-docs) を参照して、ドキュメント サイトをデプロイする方法を確認してください。

完全な dbt ドキュメント エクスペリエンスを実現するために dbt Explorer の使用にご興味がある場合は、無料の [dbt Cloud トライアル](https://www.getdbt.com/signup) にサインアップするか、[お問い合わせ](https://www.getdbt.com/contact) してください。
