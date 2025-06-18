---
title: "Data health tile"
id: "data-tile"
sidebar_label: "Data health tile"
description: "Embed data health tiles in your dashboards to distill data health signals for data consumers."
image: /img/docs/collaborate/dbt-explorer/data-tile-pass.jpg
---

# データヘルスタイル <Lifecycle status="managed,managed_plus" />

データヘルスタイルを使用すると、関係者は表示中のデータが古くなっているか劣化しているかを一目で確認できます。これにより、チームはすぐに <Constant name="explorer" /> に戻って詳細を確認し、問題を調査できます。

データヘルスタイルの特徴：

- データ利用者向けに [データヘルスシグナル](/docs/explore/data-health-signals) を抽出します。
- <Constant name="explorer" /> にディープリンクし、上流のデータの問題をさらに詳しく調査できます。
- より豊富な情報を提供し、デバッグを容易にします。
- 既存の [ジョブベースのタイル](#job-based-data-health) を刷新します。

データヘルスタイルは、[エクスポージャー](/docs/build/exposures) を利用してダッシュボードにデータヘルスシグナルを表示します。エクスポージャーとは、ダッシュボードやレポートなどの特定の出力がデータモデルにどのように依存するかを定義するものです。 dbt のエクスポージャーは、以下の 2 つの方法で設定できます。

- 手動 - プロジェクトの YAML ファイルで [手動で](/docs/build/exposures#declaring-an-exposure) 定義し、明示的に定義します。
- 自動 - サポートされている <Constant name="cloud" /> 統合で自動的にプルされます。<Constant name="cloud" /> は自動的に [ダウンストリーム エクスポージャーを作成および視覚化](/docs/cloud-integrations/downstream-exposures) するため、手動での YAML 定義は不要になります。これらのダウンストリーム エクスポージャーは、dbt のメタデータ システムに保存され、[<Constant name="explorer" />](/docs/explore/explore-projects) に表示され、手動のエクスポージャーと同様に動作しますが、YAML ファイルには存在しません。

<DocCarousel slidesPerView={1}>
<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tile-pass.jpg" width="60%" title="Example of passing Data health tile in your dashboard." />
<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tiles.png" width="60%" title="Embed data health tiles in your dashboards to distill data health signals for data consumers." />
</DocCarousel>

## 前提条件

- [エンタープライズプラン](https://www.getdbt.com/pricing/)の<Constant name="cloud" />アカウントが必要です。
- [サービストークン](/docs/dbt-cloud-apis/service-tokens#permissions-for-service-account-tokens)を設定するには、アカウント管理者である必要があります。
- [開発権限](/docs/cloud/manage-access/seats-and-users)が必要です。
- プロジェクトで[エクスポージャー](/docs/build/exposures)が定義されている必要があります。
    - 手動エクスポージャーを使用する場合は、YAMLファイルで明示的に定義する必要があります。
    - 自動ダウンストリームエクスポージャーを使用する場合は、BIツールが<Constant name="cloud" />で[構成](/docs/cloud-integrations/downstream-exposures-tableau)されていることを確認してください。
- このエクスポージャを生成するジョブで、[ソースの鮮度](/docs/deploy/source-freshness)が有効になっていること。
- データヘルスタイルに使用するエクスポージャの[`type`プロパティ](/docs/build/exposures#available-properties)が`dashboard`に設定されている必要があります。そうでない場合、dbt Explorerの[**ダッシュボードにデータヘルスタイルを埋め込む**]ドロップダウンが表示されません。

## dbt Explorer でエクスポージャーを表示する

まず、このエクスポージャーを生成するジョブで [ソース鮮度](/docs/deploy/source-freshness) が有効になっていることを確認してください。

1. ナビゲーションの [Explore] リンクをクリックして、<Constant name="explorer" /> に移動します。
2. メインの [Overview] ページで、左側のナビゲーションに移動します。
3. [Resources] タブで [Exposures] をクリックし、[exposures](/docs/build/exposures) リストを表示します。
4. ダッシュボードのエクスポージャーを選択し、[General] タブに移動してデータヘルス情報を表示します。
5. このタブには、次の情報が表示されます。
    - エクスポージャーの名前。
    - データヘルスステータス: データ鮮度合格、データ品質合格、データが古い可能性がある、データ品質低下。
    - リソースタイプ (モデル、ソースなど)。
    - ダッシュボードのステータス：失敗、合格、古い。
    - 最後に完了したチェック、最後のチェックの時刻、最後のチェックの所要時間も確認できます。
6. 右上の「ダッシュボードを開く」ボタンをクリックすると、分析ツールですぐに確認できます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tile-exposures.jpg" width="95%" title="View an exposure in dbt Explorer." />

## ダッシュボードへの埋め込み

<Constant name="explorer" /> でエクスポージャーにアクセスしたら、データヘルスタイルと [サービストークン](/docs/dbt-cloud-apis/service-tokens) を設定する必要があります。URL または iFrame の埋め込みをサポートするあらゆる分析ツールにデータヘルスタイルを埋め込むことができます。

データヘルスタイルを設定するには、以下の手順に従ってください。

1. <Constant name="cloud" /> の **アカウント設定** に移動します。
2. 左側のサイドバーで [**API トークン**] を選択し、[**サービストークン**] を選択します。
3. [**サービストークンの作成**] をクリックし、名前を付けます。
4. [**メタデータのみ**](/docs/dbt-cloud-apis/service-tokens) 権限を選択します。このトークンは、後の手順でダッシュボードにタイルを埋め込む際に使用されます。
<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tile-setup.jpg" width="95%" title="Set up your dashboard status tile and service token to embed a data health tile" />

5. **メタデータのみ** トークンをコピーし、安全な場所に保存します。このトークンは次の手順で必要になります。
6. <Constant name="explorer" /> に戻り、エクスポージャーを選択します。

   :::tip
    データヘルスタイルに使用するエクスポージャーの [`type` プロパティ](/docs/build/exposures#available-properties) が `dashboard` に設定されている必要があります。そうでない場合、dbt Explorer の「**ダッシュボードにデータヘルスタイルを埋め込む**」ドロップダウンが表示されません。
   :::

7. **データヘルス** セクションの下にあるトグルを展開すると、エクスポージャータイルを埋め込む方法の説明が表示されます（開発権限を持つアカウント管理者の場合）。
8. 展開されたトグルに、**メタデータのみのトークン** を貼り付けることができるテキストフィールドが表示されます。
<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tile-example.jpg" width="85%" title="Expand the toggle to embed data health tile into your dashboard." />

9. トークンを貼り付けたら、ダッシュボードに追加するものに応じて**URL** または **iFrame** を選択できます。

アナリティクスツールが iFrame をサポートしている場合は、ダッシュボードタイルを iFrame 内に埋め込むことができます。

## 例
次の例は、PowerBI、Tableau、Sigma にデータヘルスタイルを埋め込む方法を示しています。

<Tabs>

<TabItem value="powerbi" label="PowerBI example">

Power BI Pro Online、Fabric PowerBI、または PowerBI Desktop を使用して、データ ヘルス タイル iFrame を Power BI に埋め込むことができます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/power-bi.png" width="80%" title="Embed data health tile iFrame in PowerBI"/>

PowerBI にデータヘルスタイルを埋め込むには、以下の手順に従ってください。

1. PowerBI でダッシュボードを作成し、データベースに接続してデータを取得します。
2. **データ** を右クリックし、**その他のオプション** を選択して、**新しいメジャー** を選択して、新しい PowerBI メジャーを作成します。
<Lightbox src="/img/docs/collaborate/dbt-explorer/power-bi-measure.png" width="80%" title="Create a new PowerBI measure."/>

3. <Constant name="explorer" /> に移動し、エクスポージャーを選択して、[**ダッシュボードにデータヘルスを埋め込む**](/docs/explore/data-tile#embed-in-your-dashboard) トグルを展開します。
4. **iFrame** タブに移動し、iFrame コードをコピーします。メタデータのみのトークンが既に設定されていることを確認してください。
5. PowerBI で、コピーした iFrame コードをメジャー計算ウィンドウに貼り付けます。iFrame コードは次のようになります。

    ```html
        Website =
        "<iframe src='https://1234.metadata.ACCESS_URL/exposure-tile?uniqueId=exposure.EXPOSURE_NAME&environmentType=staging&environmentId=123456789&token=YOUR_METADATA_TOKEN' title='Exposure status tile' height='400'></iframe>"
    ```

    <Lightbox src="/img/docs/collaborate/dbt-explorer/power-bi-measure-tools.png" width="90%" title="In the 'Measure tools' tab, replace your values with the iFrame code."/>

6. PowerBI デスクトップはデフォルトでは HTML レンダリングをサポートしていないため、PowerBI Visuals Store から HTML コンポーネントをインストールする必要があります。
7. これを行うには、[**ビジュアルの作成**] に移動し、[**その他のビジュアルの取得**] を選択します。
8. PowerBI アカウントでログインします。
9. サードパーティ製の HTML ビジュアルがいくつかあります。このガイドでテストしたのは [HTML コンテンツ](https://appsource.microsoft.com/en-us/product/power-bi-visuals/WA200001930?tab=Overview) です。これをインストールしてください。ただし、これはサードパーティ製のプラグインであり、dbt Labs によって作成またはサポートされていないことにご注意ください。
10. iFrame コードを含むメトリックを PowerBI の HTML コンテンツ ウィジェットにドラッグします。これで、データ正常性タイルが表示されます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/power-bi-final.png" width="80%" title="Drag the metric with the iFrame code into the HTML content widget in PowerBI. This should now display your data health tile."/>

*Power BI レポートに Web サイトを埋め込む方法の詳細については、[このチュートリアル](https://www.youtube.com/watch?v=SUm9Hnq8Th8) を参照してください。*

</TabItem>

<TabItem value="tableau" label="Tableau example">

Tableau にデータ ヘルス タイルを埋め込むには、次の手順に従います:

<Lightbox src="/img/docs/collaborate/dbt-explorer/tableau-example.png" width="80%" title="Embed data health tile iFrame in Tableau"/>

1. Tableau でダッシュボードを作成し、データベースに接続してデータを取得します。
2. dbt Explorer の [**データヘルス**] セクションにある [**ダッシュボードにデータヘルスを埋め込む**] トグルで利用可能な URL または iFrame スニペットをコピーしたことを確認します。
3. [**Web ページ**] オブジェクトを挿入します。
4. URL を挿入し、[**OK**] をクリックします。

    ```html
    https://metadata.ACCESS_URL/exposure-tile?uniqueId=exposure.EXPOSURE_NAME&environmentType=production&environmentId=220370&token=<YOUR_METADATA_TOKEN>
    ```

    *注: プレースホルダーを実際の値に置き換えてください。*
5. Tableau ダッシュボードに埋め込まれたデータ ヘルス タイルが表示されるようになります。

</TabItem>

<TabItem value="sigma" label="Sigma example">

Sigma にデータ ヘルス タイルを埋め込むには、次の手順に従います:

<Lightbox src="/img/docs/collaborate/dbt-explorer/sigma-example.jpg" width="90%" title="Embed data health tile in Sigma"/>

1. Sigma でダッシュボードを作成し、データベースに接続してデータを取得します。
2. dbt Explorer の「**データヘルス**」セクションにある「**ダッシュボードにデータヘルスを埋め込む**」トグルで利用可能な URL または iFrame スニペットをコピーしたことを確認します。
3. Sigma ワークブックに、次の形式で新しい埋め込み UI 要素を追加します:

    ```html
    https://metadata.ACCESS_URL/exposure-tile?uniqueId=exposure.EXPOSURE_NAME&environmentType=production&environmentId=ENV_ID_NUMBER&token=<YOUR_METADATA_TOKEN>
    ```

    *注：プレースホルダーは実際の値に置き換えてください。*
4. これで、Sigma ダッシュボードにデータヘルスタイルが埋め込まれているはずです。

</TabItem>

</Tabs>

## ジョブベースのデータヘルス <Lifecycle status="Legacy"/>

デフォルトのエクスペリエンスは、<Constant name="explorer" /> を使用した [環境ベースのデータヘルス タイル](#view-exposure-in-dbt-explorer) です。

このセクションは、従来のジョブベースのデータヘルス タイルに関するものです。改良された環境ベースのエクスポージャー タイルを使用している場合は、前のセクションを参照してください。従来のジョブベースのデータヘルス タイルの詳細については、以下を展開してください。

<Expandable alt_header="ジョブベースのデータヘルス">
<Constant name="cloud" /> では、[Discovery API](/docs/dbt-cloud-apis/discovery-api) を使用して、ジョブベースのダッシュボード ステータス タイルを強化できます。ダッシュボード ステータス タイルはダッシュボード（具体的には、iFrame を埋め込むことができる場所）に配置され、ダッシュボードに送られるデータの品質と鮮度に関する分析情報を提供します。これは、dbt [exposures](/docs/build/exposures) で実行されます。

#### 機能
ダッシュボードのステータスタイルは次のようになります:

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/dashboard-status-tiles/passing-tile.jpeg"/>

エクスポージャーに入力されるデータソースのいずれかが古い場合、データ鮮度チェックは失敗します。いずれかのdbtテストが失敗した場合、データ品質チェックは失敗します。失敗状態は以下のようになります:

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/dashboard-status-tiles/failing-tile.jpeg"/>

ダッシュボード ステータス タイルから [**詳細を表示**] をクリックすると、ランディング ページに移動し、このエクスポージャーにフィードされる特定のソース、モデル、テストの詳細を確認できます。

#### セットアップ
まず、このエクスポージャーを生成するジョブで [ソースの鮮度](/docs/deploy/source-freshness) を有効にしてください。

ダッシュボードのステータスタイルを設定するには、以下のものが必要です。

1. **メタデータのみのトークン。** メタデータのみのトークンの設定方法については、[こちら](/docs/dbt-cloud-apis/service-tokens) をご覧ください。

2. **エクスポージャー名。** エクスポージャーの設定方法の詳細については、[こちら](/docs/build/exposures) をご覧ください。

3. **ジョブ ID。** <Constant name="cloud" /> で関連ジョブを表示するときに、URL から直接ジョブ ID を選択できます。

これらの 3 つのフィールドを次の iFrame に挿入し、**iFrame を埋め込むことができる場所であればどこにでも埋め込むことができます**。

```
<iframe src='https://metadata.YOUR_ACCESS_URL/exposure-tile?name=<exposure_name>&jobId=<job_id>&token=<metadata_only_token>' title='Exposure Status Tile'></iframe>
```

:::tip `YOUR_ACCESS_URL` を、ご利用のリージョンとプランのアクセス URL に置き換えてください。

<Constant name="cloud" /> は世界中の複数のリージョンでホストされており、リージョンごとにアクセス URL が異なります。`YOUR_ACCESS_URL` を、ご利用のリージョンとプランに応じた [アクセス URL](/docs/cloud/about-cloud/access-regions-ip-addresses) に置き換えてください。例えば、アカウントが EMEA リージョンでホストされている場合は、次の iFrame コードを使用します。

```
<iframe src='https://metadata.emea.dbt.com/exposure-tile?name=<exposure_name>&jobId=<job_id>&token=<metadata_only_token>' title='Exposure Status Tile'></iframe>
```

:::

#### BIツールへの埋め込み
ダッシュボードのステータスタイルは、iFrameを埋め込める場所であればどこでも機能します。以下に、一般的なBIツールとの統合方法に関するヒントをいくつかご紹介します。

<Tabs>
<TabItem value="mode" label="Mode">

#### Mode
Mode では、iFrame を埋め込んだレポートの HTML を直接 [編集](https://mode.com/help/articles/report-layout-and-presentation/#html-editor) できます。

Mode は、<Constant name="cloud" /> Discovery API との独自の [統合](https://mode.com/get-dbt/) も構築しています。
</TabItem>

<TabItem value="looker" label="Looker">

#### Looker
LookerではHTMLを直接埋め込むことができず、代わりに[カスタムビジュアライゼーション](https://docs.looker.com/admin-options/platform/visualizations)を作成する必要があります。管理者がこれを行う方法の一つは次のとおりです。
- Looker管理者向けのビジュアライゼーションページに[新しいビジュアライゼーション](https://fishtown.looker.com/admin/visualizations)を追加します。[こちらのURL](https://metadata.cloud.getdbt.com/static/looker-viz.js)を使用して、iFrameを利用したLookerビジュアライゼーションを設定できます。設定例は以下のとおりです。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/dashboard-status-tiles/looker-visualization.jpeg" title="Configure a Looker visualization powered by the iFrame" />

- カスタムビジュアライゼーションを設定したら、どのダッシュボードでも使用できます。ダッシュボードに関連するエクスポージャー名、ジョブID、トークンを使用して設定できます。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/dashboard-status-tiles/custom-looker.jpeg " width="60%"/>
</TabItem>

<TabItem value="tableau" label="Tableau">

#### Tableau
Tableau では iFrame を埋め込む必要はありません。Tableau ダッシュボード上の Web ページオブジェクトと、以下の形式の URL を使用するだけで済みます。

```
https://metadata.YOUR_ACCESS_URL/exposure-tile?name=<exposure_name>&jobId=<job_id>&token=<metadata_only_token>
```

:::tip `YOUR_ACCESS_URL` を、ご利用のリージョンとプランのアクセス URL に置き換えてください。

<Constant name="cloud" /> は世界中の複数のリージョンでホストされており、リージョンごとにアクセス URL が異なります。`YOUR_ACCESS_URL` を、ご利用のリージョンとプランに応じた [アクセス URL](/docs/cloud/about-cloud/access-regions-ip-addresses) に置き換えてください。例えば、アカウントが北米リージョンでホストされている場合は、次のコードを使用します。

```
https://metadata.cloud.getdbt.com/exposure-tile?name=<exposure_name>&jobId=<job_id>&token=<metadata_only_token>

```
:::

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/dashboard-status-tiles/tableau-object.png" width="60%" title="Configure Tableau by using a Web page object." />
</TabItem>

<TabItem value="sigma" label="Sigma">

#### Sigma

Sigma では iFrame を埋め込む必要はありません。Sigma ワークブックに、以下の形式で新しい埋め込み UI 要素を追加してください。

```
https://metadata.YOUR_ACCESS_URL/exposure-tile?name=<exposure_name>&jobId=<job_id>&token=<metadata_only_token>
```

:::tip `YOUR_ACCESS_URL` を、ご利用のリージョンとプランのアクセス URL に置き換えてください。

<Constant name="cloud" /> は世界中の複数のリージョンでホストされており、リージョンごとにアクセス URL が異なります。`YOUR_ACCESS_URL` を、ご利用のリージョンとプランに応じた [アクセス URL](/docs/cloud/about-cloud/access-regions-ip-addresses) に置き換えてください。例えば、アカウントが APAC リージョンでホストされている場合は、次のコードを使用します。

```
https://metadata.au.dbt.com/exposure-tile?name=<exposure_name>&jobId=<job_id>&token=<metadata_only_token>

```
:::

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/dashboard-status-tiles/sigma-embed.gif" width="60%" title="Configure Sigma by using an embedded UI element." />
</TabItem>
</Tabs>

</Expandable>
