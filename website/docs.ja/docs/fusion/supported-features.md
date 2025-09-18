---
title: "サポートされている機能"
id: "supported-features"
description: "Feature support and parity information for the dbt Fusion engine."
pagination_next: null
pagination_prev: null
---

# サポートされている機能

<IntroText>

要件や制限など、dbt Fusion エンジンでサポートされている機能について説明します。

</IntroText>

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';
import FusionDWH from '/snippets/_fusion-dwh.md';

<VersionBlock lastVersion="1.99">

<FusionBeta />

</VersionBlock>

### dbt Core との互換性

私たちの目標は、<Constant name="fusion_engine" /> が <Constant name="core" /> フレームワークのすべての機能をサポートし、さらに拡張することです。Fusion は既に <Constant name="core" /> v1.9 で多くの機能をサポートしており、現在、さらなる機能追加に向けて迅速に取り組んでいます。

非推奨となった機能の一部を削除し、エラーのあるプロジェクトコードの検証をより厳密に行うようになりました。詳細は、[アップグレードガイド](/docs/dbt-versions/core-upgrade/upgrading-to-fusion) をご覧ください。

## 要件

dbt プロジェクトで Fusion を使用するには、以下の条件を満たしている必要があります。
- サポートされているアダプタと認証方法を使用していること。
  <FusionDWH />
- プロジェクトでは SQL モデルのみが定義されていること。Fusion は Python モデルを解析して他のモデルへの依存関係 (refs) を抽出できないため、現在 Python モデルはサポートされていません。 <!-- [TODO: Link to dbt-fusion Python issue.] -->

### 制限事項

プロジェクトで以下の表に記載されている機能のいずれかを使用している場合、Fusion を使用できますが、以下の理由により、すべてのワークロードを完全に移行することはできません。
- 特定のマテリアライゼーション機能を利用するモデルは実行できないか、必要な構成が不足している可能性があります。
- dbt Core のログ出力をそのまま使用するツール。Fusion のログシステムは現在不安定で不完全です。
- Fusion がまだサポートしていない、dbt プラットフォームの補完的な機能（モデルレベルの通知、高度な CI、セマンティック レイヤーなど）を中心に構築されたワークフロー。

:::note
一般提供開始に先立ち、ベータ版とプレビュー期間中にこれらの機能の多くを迅速に実装してきました。詳しくは[一般提供への道筋](/blog/dbt-fusion-engine-path-to-ga)をご覧ください。また、[`dbt-fusion` マイルストーン](https://github.com/dbt-labs/dbt-fusion/milestones)で進捗状況をご確認ください。
:::

import FusionFeatures from '/snippets.ja/_fusion-missing-features.md';

<FusionFeatures />

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />

### Package support

import FusionPackages from '/snippets.ja/_fusion-supported-packages.md';

<FusionPackages />
