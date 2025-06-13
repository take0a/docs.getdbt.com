---
title: "Supported features"
id: "supported-features"
description: "Feature support and parity information for the dbt Fusion engine."
pagination_next: null
pagination_prev: null
---

# サポートされている機能 <Lifecycle status="beta" />

<IntroText>

要件や制限など、dbt Fusion エンジンでサポートされている機能について説明します。

</IntroText>

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';
import FusionDWH from '/snippets/_fusion-dwh.md';
import FusionA from '/snippets/_fusion-auth.md';

<FusionBeta />

### dbt Core との互換性

dbt Fusion エンジンが dbt Core フレームワークのすべての機能をサポートし、さらにその先へ進むことを目指しています。Fusion は既に <Constant name="core" /> v1.9 の多くの機能をサポートしており、現在も迅速な対応でさらなる機能追加に取り組んでいます。

非推奨となった機能の一部を削除し、エラーのあるプロジェクトコードの検証をより厳密に行うようになりました。詳細は、[アップグレードガイド](/docs/dbt-versions/core-upgrade/upgrading-to-fusion) をご覧ください。

## 要件

dbt プロジェクトで Fusion を使用するには、以下の要件を満たす必要があります。
- サポートされているデータウェアハウスを使用する:
  <FusionDWH />
- サポートされている認証方法を使用する:
  <FusionA />
- プロジェクトでは SQL モデルのみを定義する。Fusion は Python モデルを解析して他のモデルへの依存関係 (refs) を抽出できないため、現在 Python モデルはサポートされていません。 <!-- [TODO: Link to dbt-fusion Python issue.] -->

### 制限事項

プロジェクトで以下の表に記載されている機能のいずれかを使用している場合、Fusion を使用できますが、以下の理由により、すべてのワークロードを完全に移行することはできません。
- 特定のマテリアライゼーション機能を利用するモデルは実行できないか、必要な構成が不足している可能性があります。
- dbt Core のログ出力をそのまま使用するツール。Fusion のログシステムは現在不安定で不完全です。
- Fusion がまだサポートしていない、dbt プラットフォームの補完的な機能（モデルレベルの通知、高度な CI、セマンティック レイヤーなど）を中心に構築されたワークフロー。

:::note
ベータ期間中および一般提供開始前に、これらの機能を可能な限り迅速に実装する予定です。詳しくは[一般提供への道筋](/blog/dbt-fusion-engine-path-to-ga)をご覧ください。
:::

import FusionFeatures from '/snippets.ja/_fusion-missing-features.md';

<FusionFeatures />

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />


