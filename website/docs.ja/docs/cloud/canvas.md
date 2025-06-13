--- 
title: "About dbt Canvas" 
id: canvas      
sidebar_label: "About dbt Canvas" 
description: "dbt Canvas enables you to quickly create and visualize dbt models through a visual, drag-and-drop experience inside of dbt." 
pagination_next: "docs/cloud/canvas-interface"
pagination_prev: null
---

import Prerequisites from '/snippets.ja/_canvas-prerequisites.md';

# About Canvas <Lifecycle status='managed,managed_plus'/> 

<p style={{ color: '#717d7d', fontSize: '1.1em' }}>
<Constant name="visual_editor" /> は、視覚的なドラッグ アンド ドロップ操作と、カスタム コード生成用の組み込み AI により、データにすばやくアクセスして変換するのに役立ちます。
</p>

<Constant name="visual_editor" /> を使用すると、組織はコード駆動開発の多くのメリット（精度の向上、デバッグの容易さ、検証の容易さなど）を享受できると同時に、さまざまな開発担当者がそれぞれの環境で開発できる柔軟性も維持できます。また、組み込み AI を活用してカスタムコードを生成することで、エンドツーエンドでスムーズなエクスペリエンスを実現できます。

これらのモデルは SQL に直接コンパイルされ、プロジェクト内の他の dbt モデルと区別がつきません。
- ビジュアルモデルは、バッキング <Constant name="git" /> プロバイダーでバージョン管理されます。
- すべてのモデルは、[<Constant name="mesh" />](/best-practices/how-we-mesh/mesh-1-intro) 内のプロジェクト間でアクセスできます。
- モデルは、[<Constant name="cloud" /> オーケストレーション](/docs/deploy/deployments) を通じて本番環境に実装することも、ユーザーの開発スキーマに直接組み込むこともできます。
- [<Constant name="explorer" />](/docs/explore/explore-projects) および [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) と統合します。

<Lightbox src="/img/docs/dbt-cloud/canvas/canvas.png" width="90%" title="Create or edit dbt models with Canvas, enabling everyone to develop with dbt through a drag-and-drop experience inside of dbt." />

<Prerequisites feature={'/snippets.ja/_canvas-prerequisites.md'} />

## フィードバック

AIによって生成されたコードとコンテンツは、誤った結果を生成する可能性があるため、必ずご確認ください。<Constant name="visual_editor" /> の機能は、ベータ版トライアルの一環として追加または削除される場合があります。

フィードバックをお寄せいただくには、dbt Labsアカウントチームまでご連絡ください。<Constant name="visual_editor" /> の改善に役立ててまいりますので、皆様からのフィードバックとご提案をお待ちしております。

## リソース

Canvas について詳しくはこちら:

- [Canvas の使い方](/docs/cloud/use-canvas)
- Canvas [クイックスタートガイド](/guides/canvas)
- [dbt Learn](https://learn.getdbt.com/catalog) の [Canvas 基礎コース](https://learn.getdbt.com/learn/course/canvas-fundamentals)
