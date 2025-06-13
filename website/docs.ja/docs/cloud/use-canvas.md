---
title: "Edit and create dbt models" 
id: use-canvas    
sidebar_label: "Edit and create dbt models" 
description: "Access and use Canvas to create or edit dbt models through a visual, drag-and-drop experience inside of dbt." 
pagination_prev: "docs/cloud/canvas-interface"
pagination_next: "docs/cloud/build-canvas-copilot"
---

import Prerequisites from '/snippets.ja/_canvas-prerequisites.md';

# Edit and create dbt models <Lifecycle status='managed, managed_plus'/> 

<p style={{ color: '#717d7d', fontSize: '1.1em' }}>
<Constant name="visual_editor" /> にアクセスして使用することで、視覚的なドラッグ＆ドロップ操作で dbt モデルを作成または編集できます。開発エクスペリエンスにおいて、組み込みの AI を活用してカスタムコード生成を行うことができます。
</p>

## Canvas へのアクセス

エディターにアクセスする前に、<Constant name="cloud" /> プロジェクトがセットアップされている必要があります。これには、<Constant name="git" /> リポジトリ、データプラットフォーム接続、環境、開発者認証情報が含まれます。これらがセットアップされていない場合は、<Constant name="cloud" /> 管理者にお問い合わせください。

<Constant name="visual_editor" /> にアクセスするには:

- 左側のパネルに移動します。
- **開発** をクリックし、**<Constant name="visual_editor" />** を選択します。

<Lightbox src="/img/docs/dbt-cloud/canvas/access-canvas.png" width="80%" title="Access Canvas by selecting 'Develop' from the navigation menu." />

<Prerequisites feature={'/snippets.ja/_canvas-prerequisites.md'} />

## モデルの作成

dbt SQL モデルを作成するには、「**新しいモデルの作成**」をクリックし、以下の手順を実行してください。<Constant name="visual_editor" /> ではソースモデルを作成できないことに注意してください。これは、ソースが既に作成されている状態で本番環境を実行する必要があるためです。

1. 演算子ツールバーから演算子をドラッグし、キャンバスにドロップします。
2. 演算子をクリックして、設定パネルを開きます。
	#### 入力
		- **入力モデル**: 使用するモデルと列を選択します。
	<br />
	#### 変換
		- **結合**: 結合条件を定義し、両方のテーブルから列を選択します。
		- **選択**: モデルから必要な列を選択します。
		- **集計**: 集計関数と、それらが適用される列を指定します。
		- **数式**: 新しい列を作成するための数式を追加します。疑問符 (?) アイコンをクリックすると、組み込みの AI コードジェネレーターを使用して SQL コードを生成することができます。プロンプトを入力し、結果が表示されるまで待ちます。
		- **フィルター**: データをフィルタリングする条件を設定します。
		- **順序**: 並べ替えの基準となる列と並べ替え順序を選択します。
		- **制限**: 返す行の最大数を設定します。
		<br />
	#### 出力モデル
		- **出力モデル**: dbt モデルによって生成された最終的な変換済みデータセット。

	現在、<Constant name="visual_editor" /> では出力モデルを 1 つしか使用できませんが、将来的には複数の出力モデルを使用できるようになる予定です。
3. **出力** タブと **SQL コード** タブを表示します。
		- 各演算子には出力タブがあり、構成済みのノードからのデータをプレビューできます。
		- コードタブには、ノードの構成によって生成された SQL コードが表示されます。これを使用して、ビジュアルモデル構成の SQL を確認します。
4. コネクタを使用して演算子を接続します。演算子の「+」開始点と接続先の演算子の間をカーソルでドラッグし、接続先の演算子にリンクします。これでコネクタラインが作成されます。
		- これにより、ソーステーブルから、設定したさまざまな変換を経て最終出力にデータが流れます。
5. dbtモデルの構築を続け、**出力**タブで出力を確認します。

<!-- 
### Configure nodes
- Built-in AI code generator

### View output
-->

## 既存のモデルを編集する

既存のモデルを編集するには、<Constant name="visual_editor" /> ワークスペースに移動し、左上の**モデルアイコン** ボタンをクリックし、**+** をクリックして、**既存のモデルを編集** をクリックします。これにより、編集するモデルを選択できます。

<Lightbox src="/img/docs/dbt-cloud/canvas/edit-model.png" width="90%" title="Edit a model using the 'Edit a model' button." />

## バージョン管理

モデルのテストとドキュメント作成は、開発プロセスにおいて重要な部分です。

乞うご期待！近日中に、<Constant name="visual_editor" /> で dbt モデルのバージョン管理ができるようになります。これにより、変更内容を追跡し、必要に応じて以前のバージョンに戻すことが可能になります。

<!-- leaving this section here in case there's more to add later if needed
## Limitations
Are there limitations here?
-->
