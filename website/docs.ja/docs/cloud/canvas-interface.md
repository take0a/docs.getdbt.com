--- 
title: "Navigate the interface" 
id: canvas-interface      
sidebar_label: "Navigate the interface" 
description: "The dbt Canvas interface contains an operator toolbar, operators, and a canvas to help you access and transform data through a seamless drag-and-drop dbt model creation experience in dbt." 
pagination_next: "docs/cloud/use-canvas"
pagination_prev: "docs/cloud/canvas"

---

# インターフェースを操作する <Lifecycle status='managed, managed_plus'/> 

<p style={{ color: '#717d7d', fontSize: '1.1em' }}>

<Constant name="visual_editor" /> インターフェースには、演算子ツールバー、演算子、キャンバス、組み込み AI などが含まれており、<Constant name="cloud" /> でのシームレスなドラッグ アンド ドロップによる dbt モデル作成エクスペリエンスを通じてデータにアクセスし、変換するのに役立ちます。

</p>

このページでは、ユーザーインターフェース要素の包括的な定義と用語を提供し、<Constant name="visual_editor" /> のインターフェースを簡単に操作できるようにします。

<Constant name="visual_editor" /> インターフェースは、以下の要素で構成されています。

- **演算子ツールバー** &mdash; インターフェース上部にあるツールバーには、利用可能なすべてのノードカテゴリが表示されます。
	- 入力
	- 変換
	- 出力
- **演算子** &mdash; ソースデータの提供、特定の変換の実行、レイヤー設定（モデル、結合、集計、フィルターなど）を行うタイルです。コネクタを使用して演算子をリンクし、完全なデータ変換パイプラインを構築します。
- **キャンバス** &mdash; ノードツールバーの下にあるメインのホワイトボード領域です。キャンバスでは、洗練されたドラッグアンドドロップ操作でモデルを作成または変更できます。
- **設定パネル** &mdash; 各演算子には、クリックすると開く設定パネルがあります。構成パネルでは、演算子を構成したり、現在のモデルを確認したり、テーブルへの変更をプレビューしたり、ノードの SQL コードを表示したり、演算子を削除したりできます。

## 演算子

キャンバス上部の演算子ツールバーには、利用可能な様々な変換演算子が表示されます。各演算子を使用して、フィルターの追加や、演算子をキャンバスにドラッグしてモデルを結合するなど、特定のタスクを設定または実行できます。コネクタラインを使用して演算子を接続することで、データ変換のための完全なモデルを作成できます。

<Lightbox src="/img/docs/dbt-cloud/canvas/operators.png" width="90%" title="Use the operator toolbar to perform different transformation operations." />

ここでは次の演算子が使用できます:

#### 入力

入力演算子はソースデータを設定します。
- **入力モデル**: 使用するモデルと列を選択します。

#### 変換

変換演算子はデータを整形します。
- **結合**: 結合条件を定義し、両方のテーブルから列を選択します。
- **選択**: モデルから必要な列を選択します。
- **集計**: 集計関数と、それらが適用される列を指定します。
- **数式**: 新しい列を作成するための数式を追加します。疑問符 (?) アイコンをクリックすると、組み込みの AI コードジェネレーターを使用して SQL コードが生成されます。プロンプトを入力し、結果が表示されるまで待ちます。
- **フィルター**: データをフィルターするための条件を設定します。
- **順序**: 並べ替える列と並べ替え順序を選択します。
- **制限**: 返す行の最大数を設定します。
    
#### 出力モデル

出力演算子は、変換されたデータの名前と場所を設定します。
- **出力モデル**: モデルによって生成された最終的な変換済みデータセット。

現在、<Constant name="visual_editor" /> では出力モデルを1つしか使用できませんが、将来的には複数の出力モデルを使用できるようになる予定です。

各演算子をクリックすると、設定パネルが開きます。設定パネルでは、演算子の設定、現在のモデルの確認、モデルへの変更のプレビュー、ノードのSQLコードの表示、演算子の削除を行うことができます。

<Lightbox src="/img/docs/dbt-cloud/canvas/canvas.png" width="90%" title="The Canvas interface that contains a node toolbar and canvas." />

追加の演算子に関するご意見がございましたら、ぜひお聞かせください。dbt Labs アカウントチームまでご連絡いただき、ご意見をお聞かせください。

## Canvas

<Constant name="visual_editor" /> は、dbt SQL モデルの作成と変更に便利なドラッグ＆ドロップ インターフェースを備えています。信頼できるデータを簡単に表示・配信できるデジタルホワイトボードのような空間です。キャンバスでは以下の操作が可能です。

- 演算子をドラッグ＆ドロップしてモデルを作成・設定する
- 内蔵 AI ジェネレータを使用して SQL コードを生成
- ズームインまたはズームアウトして視覚的に分かりやすく表示する
- dbt モデルのバージョン管理
- [近日公開] 作成したモデルのテストとドキュメント化

<Lightbox src="/img/docs/dbt-cloud/canvas/operators.png" width="90%" title="The operator toolbar allows you to select different nodes to configure or perform specific tasks, like adding filters or joining models." />

### コネクタ

コネクタを使用すると、演算子を接続してdbtモデルを作成できます。キャンバスに演算子を追加したら、次の操作を行います。
- 演算子の横にある「+」記号にマウスポインタを合わせてクリックします。
- 演算子の「+」開始点と接続先のノードの間をカーソルでドラッグします。コネクタラインが作成されます。
- 例えば、結合を作成するには、一方の演算子を「L」（左）に、もう一方の演算子を「R」（右）に接続します。エンドポイントは演算子の左側にあるため、コネクタをエンドポイントまで簡単にドラッグできます。

<Lightbox src="/img/docs/dbt-cloud/canvas/connector.png" width="100%" title="Click and drag your cursor to connect operators." />

## 設定パネル

各演算子には、クリックすると開く設定サイドパネルがあります。設定パネルでは、演算子の設定、現在のモデルの確認、変更のプレビュー、演算子のSQLコードの表示、演算子の削除を行うことができます。

設定サイドパネルには以下の項目があります。
- 設定タブ - このセクションでは、組み込みのAIコードジェネレータを使用してSQLを生成するなど、特定の要件に合わせて演算子を設定できます。
- 入力タブ - このセクションでは、現在のソーステーブルのデータを表示できます。モデル演算子では使用できません。
- 出力タブ - このセクションでは、変更されたソースモデルのデータをプレビューできます。
- コード - このセクションでは、データ変換の基盤となるSQLコードを表示できます。

<Lightbox src="/img/docs/dbt-cloud/canvas/config-panel.png" width="90%" title="A sleek drag-and-drop canvas interface that allows you to create or modify dbt SQL models." />
