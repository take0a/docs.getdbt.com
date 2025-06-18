---
title: "Project recommendations"
sidebar_label: "Project recommendations"
description: "dbt Explorer provides recommendations that you can take to improve the quality of your dbt project."
---

# プロジェクトの推奨事項 <Lifecycle status="managed,managed_plus" />
 
dbt Explorer は、[Discovery API](/docs/dbt-cloud-apis/discovery-api) のメタデータを使用して、`dbt_project_evaluator` [パッケージ](https://hub.getdbt.com/dbt-labs/dbt_project_evaluator/latest/) からプロジェクトに関する推奨事項を提供します。

- <Constant name="explorer" /> はグローバルビューも提供し、プロジェクト全体のすべての推奨事項が表示されるため、並べ替えや要約が容易です。
- これらの推奨事項は、より適切に文書化され、より適切にテストされ、より適切に構築された dbt プロジェクトを作成する方法に関する洞察を提供し、信頼性を高め、混乱を軽減します。
- シームレスで一貫したエクスペリエンスを実現するために、推奨事項は `dbt_project_evaluator` の事前定義された設定を使用し、パッケージまたはプロジェクトに適用されたカスタマイズはインポートしません。

import ExplorerCourse from '/snippets.ja/_explorer-course-link.md';

<ExplorerCourse />

## 推奨事項ページ
推奨事項の概要ページには、プロジェクト内のモデルのテストとドキュメントのカバレッジを測定する2つのトップレベル指標が含まれています。

- **モデルテストカバレッジ** - プロジェクト内のモデル（パッケージから取得されたモデルや <Constant name="mesh" /> 経由でインポートされたモデル以外）のうち、少なくとも1つの dbt テストが構成されているモデルの割合。
- **モデルドキュメントカバレッジ** - プロジェクト内のモデル（パッケージから取得されたモデルや <Constant name="mesh" /> 経由でインポートされたモデル以外）のうち、説明が設定されているモデルの割合。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-recommendations-overview.png" width="100%" title="Example of the Recommendations overview page with project metrics and the recommendations for all resources in the project"/>

## ルール一覧
次の表は、`dbt_project_evaluator` [パッケージ](https://hub.getdbt.com/dbt-labs/dbt_project_evaluator/latest/) で現在定義されているルールの一覧です。

| Category | Name | Description | Package Docs Link |
| --- | --- | --- | --- |
| Modeling | Direct Join to Source | モデルとソースの両方を結合するモデル。ステージング モデルが欠落していることを示します。 | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/modeling/#direct-join-to-source) |
| Modeling | Duplicate Sources | 複数のソースノードが同じデータウェアハウス関係に対応します | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/modeling/#duplicate-sources) |
| Modeling | Multiple Sources Joined  | 複数のソース親を持つモデル。ステージング モデルが不足していることを示します。 | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/modeling/#multiple-sources-joined) |
| Modeling | Root Model | 親を持たないモデル。ハードコードされた参照の可能性とソースの必要性を示唆している。  | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/modeling/#root-models) |
| Modeling | Source Fanout | 複数のモデルの子を持つソース。ステージングモデルが必要であることを示しています。 | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/modeling/#source-fanout) |
| Modeling | Unused Source | どのリソースからも参照されていないソース | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/modeling/#unused-sources) |
| Performance | Exposure Dependent on View | 少なくとも 1 つのモデル親がビューとしてマテリアライズドされているエクスポージャー。潜在的なクエリ パフォーマンスの問題を示しています。 | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/performance/#exposure-parents-materializations) |
| Testing | Missing Primary Key Test | モデルの粒度に関するテストが不十分なモデル。 | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/testing/#missing-primary-key-tests) |
| Documentation | Undocumented Models | モデルレベルの説明のないモデル | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/documentation/#undocumented-models) |
| Documentation | Undocumented Source | 説明のないソース（ソーステーブルのコレクション） | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/documentation/#undocumented-sources) |
| Documentation | Undocumented Source Tables | 説明のないソーステーブル | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/documentation/#undocumented-source-tables) |
| Governance | Public Model Missing Contract | データ型を保証するモデル契約を持たないパブリックアクセスのモデル | [GitHub](https://dbt-labs.github.io/dbt-project-evaluator/0.8/rules/governance/#public-models-without-contracts) |


## 「推奨事項」タブ

モデル、ソース、エクスポージャーのそれぞれのリソース詳細ページには、「推奨事項」タブがあり、それぞれのリソースに対応する具体的な推奨事項が表示されます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/example-recommendations-tab.png" width="80%" title="Example of the Recommendations tab "/>

