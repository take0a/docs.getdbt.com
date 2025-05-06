---
title: "モデルガバナンスについて"
id: about-model-governance
description: "モデルガバナンスに関連する新機能に関する情報"
pagination_next: "docs/collaborate/govern/model-access"
pagination_prev: null
---

[**Model access**](model-access): モデルの中には、成熟した再利用可能なデータ製品もあります。また、チームが実装を進めている途中のモデルもあります。モデルを「パブリック」または「プライベート」としてマークすることで、モデル間の区別を明確にし、誰がモデルを `ref` できるかを制御できます。

[**Model contracts**](model-contracts): 構築中のモデルの形状を保証することで、下流のクエリで予期せぬ結果や互換性を破る変更を回避します。列名、データ型、制約（データプラットフォームでサポートされているもの）を明示的に定義します。

[**Model versions**](model-versions): 互換性を破る変更が避けられない場合は、モデルの新しいバージョンを作成して、よりスムーズなアップグレードパスを提供します。これらのモデルバージョンは共通の参照名を共有し、プロパティと構成を再利用できます。

[**プロジェクト依存関係**](/docs/collaborate/govern/project-dependencies): <Lifecycle status='enterprise'/> プロジェクト名を含む [2 つの引数を持つ ref](/reference/dbt-jinja-functions/ref#ref-project-specific-models) を使用して、dbt プロジェクト間で公開モデルを参照するには、プロジェクト間依存関係を使用します。

import ModelGovernanceRollback from '/snippets/_model-governance-rollback.md';

<ModelGovernanceRollback />
