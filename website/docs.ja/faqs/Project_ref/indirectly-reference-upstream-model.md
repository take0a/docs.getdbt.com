---
title: 間接的に参照される上流のパブリック モデルがエクスプローラーに表示されないのはなぜですか?
sidebar_label: 間接的に参照される上流モデル
id: indirectly-reference-upstream-model
description: 間接的に参照されている上流のパブリックモデルがエクスプローラーに表示されない理由を説明します。
---

<Constant name="mesh" /> の [プロジェクト依存関係](/docs/mesh/govern/project-dependencies) について、[<Constant name="explorer" />](/docs/explore/explore-multiple-projects) には、上流プロジェクトから直接参照されている [パブリックモデル](/docs/mesh/govern/model-access) のみが表示されます。上流モデルが別のパブリックモデルに間接的に依存している場合でも同様です。

たとえば、次の場合:

- `project_b` が `project_a` を依存関係として追加する
- `project_b` のモデル `downstream_c` が `project_a.upstream_b` を参照する
- `project_a.upstream_b` が別のパブリックモデル `project_a.upstream_a` を参照する

次のようになります:

- Explorer には、直接参照されているパブリックモデル (この場合は `upstream_b`) のみが表示されます。
- ただし、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) の系統ビューでは、<Constant name="cloud" /> が依存関係グラフ全体を動的に解決するため、`upstream_a` (間接依存関係) が表示されます。

この動作により、<Constant name="explorer" />には特定のプロジェクトで利用可能な直接依存関係のみが表示されます。
