---
title: "Access the dbt Insights interface"
description: "Learn how to access the dbt Insights interface and run queries"
sidebar_label: "Access and run queries"
tags: [dbt Insights]
image: /img/docs/dbt-insights/insights-chart.jpg
---

# Access the dbt Insights interface <Lifecycle status="preview,managed,managed_plus" />

<IntroText>
<Constant name="query_page" /> にアクセスし、クエリを実行して結果を表示する方法を学習します。
</IntroText>

:::tip
<Constant name="query_page" /> は、Enterprise アカウント向けにプライベートベータ版としてご利用いただけます。ご参加いただくには、担当のアカウントマネージャーまでお問い合わせください。
:::

<Constant name="query_page" /> は、エディターナビゲーションを備えた豊富なコンソールエクスペリエンスを提供します。 <Constant name="query_page" /> を使用すると、次のことが可能になります。
- 複数のタブを開くオプションを使用して、SQL クエリを記述できます。
- SQL + dbt のオートコンプリート候補と構文のハイライト表示を利用できます。
- SQL クエリをブックマークできます。
- **結果** タブまたは **詳細** タブを使用して、クエリの結果とその詳細を表示できます。
- **チャート** タブを使用して、クエリ結果を視覚化できます。
- **クエリ履歴** タブを使用して、クエリの履歴とそのステータス（成功、エラー、保留など）を表示できます。
- <Constant name="copilot" /> を使用して、自然言語プロンプトを使用して SQL クエリを生成または編集できます。
- [<Constant name="copilot" />](/docs/cloud/dbt-copilot)、[<Constant name="explorer" />](/docs/explore/explore-projects)、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud)、および [<Constant name="visual_editor" />](/docs/cloud/canvas) を使用することで、データ探索、AI 支援による書き込み、コラボレーションのためのシームレスなエクスペリエンスが提供されます。

## dbt Insights インターフェースにアクセスします。

<Constant name="query_page" /> にアクセスする前に、[前提条件](/docs/explore/dbt-insights#prerequisites) が満たされていることを確認してください。

1. <Constant name="query_page" /> にアクセスするには、ナビゲーションサイドバーで [Insights] オプションを選択します。
2. [開発者認証情報](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud#get-started-with-the-cloud-ide) が設定されていない場合は、<Constant name="query_page" /> から設定を求めるメッセージが表示されます。データのクエリを実行するには、開発者認証情報に応じたウェアハウスプロバイダーの権限が必要です。
3. 認証情報の設定が完了すると、プロジェクト内の既存のモデルに対して、<Constant name="query_page" /> エディターで SQL クエリを記述、実行、編集できるようになります。

## クエリの実行

<Constant name="query_page" /> でクエリを実行するには、以下を使用できます。
- 標準SQL
- Jinja ([`ref`](/reference/dbt-jinja-functions/ref)、[`source`](/reference/dbt-jinja-functions/source) 関数、およびその他のJinja関数)
- SQLコード `ref` から対応するエクスプローラーページへのリンク
- <Term id="cte">CTE</Term> と <Term id="subquery">サブクエリ</Term>
- 基本的な集計と結合
- <Constant name="semantic_layer" /> Jinja関数を使用した <Constant name="semantic_layer" /> クエリ

## 例

<Constant name="query_page" /> でクエリを実行する方法を例を使って説明しましょう。

- [Jaffle Shop](https://github.com/dbt-labs/jaffle-shop) は、ユニークな注文数とユニークな顧客数をカウントし、素晴らしい Jaffle ショップビジネスを世界各地に展開できるかどうかを把握したいと考えています。
- このロジックを SQL で表現するために、あなた（このプロジェクトに割り当てられたアナリスト）は、年間の傾向を把握し、展開の意思決定に役立てたいと考えています。ユニーク顧客数、都市数、および総注文収益を計算する次の SQL クエリを記述してください:
<br /><br />
    ```sql
    with 

    orders as (
        select * from {{ ref('orders') }}
    ),

    customers as (
        select * from {{ ref('customers') }}
    )

    select 
        date_trunc('year', ordered_at) as order_year,
        count(distinct orders.customer_id) as unique_customers,
        count(distinct orders.location_id) as unique_cities,
        to_char(sum(orders.order_total), '999,999,999.00') as total_order_revenue
    from orders
    join customers
        on orders.customer_id = customers.customer_id
    group by 1
    order by 1
    ```

### dbt Copilot を使用する
作業を簡単にするために、[<Constant name="copilot" />](/docs/cloud/use-dbt-copilot#build-queries) を使用して時間を節約し、データ分析の他の方法を検討してください。<Constant name="copilot" /> を使用すると、プロンプトに基づいてクエリをすばやく更新したり、新しいクエリを生成したりできます。

1. クエリコンソールのサイドバーにある **<Constant name="copilot" />** アイコンをクリックして、プロンプトボックスを開きます。
2. 自然言語でプロンプトを入力し、ユニーク顧客数と総収益の年間内訳を尋ねます。[送信] をクリックします。
3. <Constant name="copilot" /> は次の応答を返します。
    - クエリの概要
    - ロジックの説明
    - 生成されたSQL
    - 既存のクエリを生成されたSQLで**追加**または**置換**するオプション
4. 出力を確認し、「**置換**」をクリックして、<Constant name="copilot" /> によって生成されたSQLをエディターで使用します。
5. 次に、「**実行**」をクリックして結果をプレビューします。

<Lightbox src="/img/docs/dbt-insights/insights-copilot.png" width="95%" title="dbt Insights with dbt Copilot" />

ここから、次の操作を実行できます。
- <Constant name="copilot" /> を使用してクエリの構築または変更を続行します。
- **[結果]** タブで [結果](#view-results) を確認します。
- **[詳細]** タブで [メタデータとクエリの詳細を表示](#view-details) します。
- **[チャート]** タブで [結果を視覚化](#chart-results) します。
- **[クエリ履歴](#query-history) でステータスと過去の実行を確認します。
- [**<Constant name="explorer" />**](#use-dbt-explorer) を使用して、モデルの系統とコンテキストを調べます。
- クエリを保存する場合は、[クエリ コンソール メニュー](/docs/explore/navigate-dbt-insights#query-console-menu) で [ブックマーク] をクリックして、後で参照できるように保存できます。

:::tip クエリをモデルに変換してみませんか？
[クエリ コンソール メニュー](/docs/explore/navigate-dbt-insights#query-console-menu)から [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) または [<Constant name="visual_editor" />](/docs/cloud/canvas) にアクセスして、SQL を再利用可能な dbt モデルに変換できます。すべて <Constant name="cloud" /> 内で実行できます。
:::

### 結果の表示

同じ例を使用して、クエリを実行し、以下の操作を行うことで探索的なデータ分析を行うことができます。

- **結果** タブで結果を表示 - クエリの結果がページ分けされて表示されます。
- 結果の並べ替え - 列ヘッダーをクリックすると、その列で結果を並べ替えることができます。
- CSV にエクスポート - 表の右上にあるダウンロードボタンをクリックして、データセットをエクスポートします。
<Lightbox src="/img/docs/dbt-insights/insights-export-csv.png" width="95%" title="dbt Insights Export to CSV" />

### 詳細の表示
**詳細** タブをクリックすると、クエリの詳細が表示されます。
- **クエリメタデータ** - <Constant name="copilot" /> によって生成されたタイトルと説明、指定されたSQL、および対応するコンパイル済みSQL。
- **接続の詳細** - 関連するデータプラットフォームの接続情報。
- **クエリの詳細** - クエリの実行時間、ステータス、列数、行数。

<Lightbox src="/img/docs/dbt-insights/insights-details.png" width="95%" title="dbt Insights Details tab" />

### チャート結果

**チャート** タブをクリックすると、クエリのチャート結果を視覚化できます。次の操作が可能です。
- チャートアイコンを使用して、チャートの種類を選択します。
- **折れ線グラフ、棒グラフ、散布図** から選択します。
- **チャート設定** アイコンを使用して、視覚化する軸と列を選択します。

<Lightbox src="/img/docs/dbt-insights/insights-chart.png" width="95%" title="dbt Insights Chart tab" />

### クエリ履歴

**クエリ履歴** アイコンを使用して、クエリの履歴とそのステータス（すべて、成功、エラー、保留中）を表示します。
- 再実行するクエリを選択して結果を表示します。
- 過去のクエリを検索し、ステータスでフィルタリングします。
- クエリにマウスオーバーすると、SQL コードを表示またはコピーできます。

クエリ履歴は無期限に保存されます。

<Lightbox src="/img/docs/dbt-insights/insights-query-history.png" width="95%" title="dbt Insights Query history icon" />

### dbt Explorer の使用

<Constant name="query_page" /> から [<Constant name="explorer" />](/docs/explore/explore-projects) に直接アクセスして、モデル、列、メトリック、ディメンションなどのプロジェクトリソースを表示します。これらはすべて <Constant name="query_page" /> インターフェースに統合されています。

この統合ビューを使用すると、開発者とユーザーはクエリワークフローを維持しながら、モデル、セマンティックモデル、メトリック、マクロなどに関する詳細なコンテキストを取得できます。統合された <Constant name="explorer" /> ビューには、次の機能が備わっています。
- <Constant name="explorer" /> と同じ検索機能
- ユーザーは表示されるオブジェクトをタイプ別に絞り込むことができます
- SQL コード `ref` から対応する Explorer ページへのハイパーリンク
- アセットを完全な <Constant name="explorer" /> エクスペリエンスで開くか、<Constant name="copilot" /> で開くことで、より詳細な情報を表示できます。

<Constant name="explorer" /> にアクセスするには、[クエリ コンソールのサイドバー メニュー](/docs/explore/navigate-dbt-insights#query-console-sidebar-menu) の **<Constant name="explorer" />** アイコンをクリックします。

<Lightbox src="/img/docs/dbt-insights/insights-explorer.png" width="90%" title="dbt Insights integrated with dbt Explorer" />

## クエリをブックマーク

Insights には、よく使うクエリをすばやく見つけられる強力なブックマーク機能があります。また、他の dbt ユーザーとブックマークを共有（または共有してもらう）オプションもあります。クエリ内のブックマークアイコンをクリックすると、リストに追加されます。

- 右側のメニューにある **ブックマークアイコン** をクリックすると、ブックマークしたクエリを管理できます。個人用クエリと共有クエリを表示できます。

    <Lightbox src="/img/docs/dbt-insights/manage-bookmarks.png" width="90%" title="Manage your query bookmarks" />
    
- **概要** タブで、説明や作成日などのブックマークの詳細を確認できます。
- **バージョン履歴** タブで、ブックマークの履歴を確認できます。バージョンをクリックすると、現在のバージョンと比較し、変更内容を確認できます。

## 考慮事項
- <Constant name="query_page" /> は開発認証情報を使用してクエリを実行します。開発認証情報を使用してアクセスできるデータウェアハウス内の任意のオブジェクトに対してクエリを実行できます。
- すべての Jinja 関数は [`defer --favor-state`](/reference/node-selection/defer) を使用して Jinja を解決します。
- 近日公開予定: `refs` を解決するために使用する環境を選択できるようになります。

<!-- this can move to another page -->

## FAQs
- <Constant name="query_page" /> と <Constant name="explorer" /> の違いは何ですか？
    - 素晴らしい質問ですね！<Constant name="explorer" /> は、データのコンテキストを提供することで、dbt プロジェクトの構造、リソース、系統、指標を理解するのに役立ちます。
    - <Constant name="query_page" /> はそのコンテキストに基づいて、<Constant name="cloud" /> 内で直接 SQL クエリを記述、実行、反復処理できるようにします。アドホック分析や探索的分析向けに設計されており、ビジネス ユーザーとアナリストがデータを探索し、質問し、シームレスに共同作業を行うことを可能にします。
    - <Constant name="explorer" /> はコンテキストを提供し、<Constant name="query_page" /> はアクションを可能にします。
