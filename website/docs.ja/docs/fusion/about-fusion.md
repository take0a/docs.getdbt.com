---
title: "About the dbt Fusion engine"
id: "about-fusion"
description: "Fusion is the next-generation engine for dbt."
pagination_next: null
pagination_prev: null
---

# About the dbt Fusion engine <Lifecycle status="beta" />

<IntroText>

dbtはデータ変換の業界標準です。dbt Fusionエンジンにより、dbtはかつてないスピードとスケールを実現します。
</IntroText>

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';

<FusionBeta />

dbt Fusion エンジンは、<Constant name="core" /> と同じデータ変換を作成するための使い慣れたフレームワークを共有し、データ開発者が作業を高速化し、変換ワークロードをより効率的に展開できるようにします。

### Fusion とは

Fusion は、<Constant name="core" /> (Python) とは異なるプログラミング言語 (Rust) で記述された、全く新しいソフトウェアです。Fusion は <Constant name="core" /> よりも大幅に高速で、複数のエンジン方言にわたる SQL をネイティブに理解します。Fusion は、最終的には dbt Core フレームワーク全体 (dbt Core の機能のスーパーセット) と、既存の dbt プロジェクトの大部分をサポートする予定です。

Fusion には、ソースコードが公開されているもの、独自のもの、そしてオープンソースのコードが混在しています。つまり、
- dbt Labs は、ソースコードの多くを [`dbt-fusion` リポジトリ](https://github.com/dbt-labs/dbt-fusion) で公開しており、ユーザーはそこでコードを閲覧したり、コミュニティのディスカッションに参加したりできます。
- Fusion の一部の機能は、クラウドベースの [dbt プラットフォーム](https://www.getdbt.com/signup) の有料ユーザーのみ利用可能です。詳細については、[サポートされている機能](/docs/fusion/supported-features#paid-features) を参照してください。

dbt Fusion エンジンのライセンスの詳細については、[こちら](http://www.getdbt.com/licenses-faq) をご覧ください。

## Fusion を使用する理由

開発者にとって、Fusion は以下のことを可能にします。
- dbt モデル内の不正な SQL を即座に検出
- インライン <Term id="cte">CTE</Term> をプレビューしてデバッグを高速化
- dbt プロジェクト全体のモデルと列定義をトレース

これらすべてに加え、さらに多くの機能が [VSCode 用 dbt 拡張機能](/docs/about-dbt-extension) で利用可能であり、その基盤として Fusion が使用されています。

Fusion は、大規模な DAG のデプロイメントをより効率的に行うこともできます。どの列がどこで使用されているか、どのソーステーブルに新しいデータがあるかを追跡することで、Fusion は新しいデータを処理する必要がある場合にのみモデルを再構築できます。この [状態認識オーケストレーション](https://docs.getdbt.com/docs/deploy/state-aware-about) は、dbt プラットフォームの機能です。

### Fusion の使い方

以下の手順を実行できます。
- [dbt プラットフォームのドロップダウン/トグル](/docs/dbt-versions/upgrade-dbt-version-in-cloud#dbt-fusion-engine) から Fusion を選択します。
- [VSCode 用の dbt 拡張機能をインストール](/docs/install-dbt-extension)
- [Fusion CLI をインストール](/docs/fusion/install-fusion)

[クイックスタート](/guides/fusion) に進んで、Fusion をすぐにお試しください。

## 今後の予定

dbt Labs は、2025年5月28日に dbt Fusion エンジンをパブリックベータ版としてリリースしました。[Fusion の一般提供開始](https://docs.getdbt.com/blog/dbt-fusion-engine-path-to-ga) に先立ち、<Constant name="core" /> と完全に同等の機能を実現する予定です。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />
