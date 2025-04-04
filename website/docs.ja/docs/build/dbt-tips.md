---
title: "dbt のヒントとコツ"
description: "Check out any dbt-related tips and tricks to help you work faster and be more productive."
sidebar_label: "dbt のヒントとコツ"
pagination_next: null
---

このページでは、dbt エクスペリエンスを向上させるための貴重な洞察と実用的なアドバイスを提供しています。dbt を初めて使用する方でも、経験豊富なユーザーでも、これらのヒントは、より効率的かつ効果的に作業するのに役立つように設計されています。

次のヒントは、次のカテゴリに分類されています:

- [パッケージのヒント](#package-tips)はワークフローの効率化に役立ちます。
- [高度なヒントとテクニック](#advanced-tips-and-techniques)は、dbt を最大限に活用するのに役立ちます。


dbt Cloud IDE を使用して開発している場合は、[キーボード ショートカット](/docs/cloud/dbt-cloud-ide/keyboard-shortcuts) ページを参照して、開発の生産性を高め、すべてのユーザーにとって簡単にすることができます。

## パッケージのヒント {#package-tips}

これらの dbt パッケージを活用してワークフローを効率化します:

| Package | Description |
|---------|-------------|
| [`dbt_codegen`](https://hub.getdbt.com/dbt-labs/codegen/latest/) |このパッケージを使用すると、モデルとソースの YML ファイルとステージング モデルの SQL ファイルを生成することができます。 |
| [`dbt_utils`](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) | パッケージには、日常の開発に役立つマクロが含まれています。たとえば、`date_spine` は、パラメータとして指定された日付間のすべての日付を含むテーブルを生成します。 |
| [`dbt_project_evaluator`](https://hub.getdbt.com/dbt-labs/dbt_project_evaluator/latest) | このパッケージは、dbt プロジェクトをベスト プラクティスのリストと比較し、モデルを更新する方法に関する提案とガイドラインを提供します。 |
| [`dbt_expectations`](https://hub.getdbt.com/calogica/dbt_expectations/latest) | パッケージには、dbt に組み込まれているテスト以外にも多くのテストが含まれています。 |
| [`dbt_audit_helper`](https://hub.getdbt.com/#:~:text=adwords-,audit_helper,-codegen) | このパッケージを使用すると、2 つのクエリの出力を比較できます。既存のロジックをリファクタリングして、新しい結果が同一であることを確認するときに使用します。 |
| [`dbt_artifacts`](https://hub.getdbt.com/brooklyn-data/dbt_artifacts/latest) | このパッケージは、dbt 実行に関する情報をデータ プラットフォームに直接保存するため、時間の経過に伴うモデルのパフォーマンスを追跡できます。 |
| [`dbt_meta_testing`](https://hub.getdbt.com/tnightengale/dbt_meta_testing/latest) | このパッケージは、dbt プロジェクトが十分にテストされ、文書化されているかどうかを確認します。 |

## 高度なヒントとテクニック {#advanced-tips-and-techniques}

- フォルダー構造を主なセレクター方法として使用します。`dbt build --select marts.marketing` は、すべてのモデルにタグを付けるよりもシンプルで回復力が高くなります。
- ビルド ケイデンスと SLA の観点からジョブについて考えます。時間ごと、日ごと、または週ごとのビルド ケイデンスを持つモデルを一緒に実行します。
- レコードのサブセットに対するアサーションをテストするには、テストに [where config](/reference/resource-configs/where) を使用します。
- [store_failures](/reference/resource-configs/store_failures) を使用すると、テストの失敗の原因となったレコードを調べることができるため、必要に応じてデータを修復したり、テストを変更したりできます。
- [重大度](/reference/resource-configs/severity) しきい値を使用して、テストの許容失敗数を設定します。
- 増分モデル構成で [incremental_strategy](/docs/build/incremental-strategy) を使用して、データの量と一意のキーの信頼性に応じて最も効果的な動作を実装します。
- `dbt_project.yml` で `vars` を設定して、特定の条件のグローバル デフォルトを定義します。その後、コマンドで `--vars` フラグを使用してこれを上書きできます。
- Jinja の [for ループ](/guides/using-jinja?step=3) を使用して、同じ変換と命名パターンを適用する必要がある一連の列を選択するなどの反復ロジックを <Term id="dry">DRY</Term> 化します。
- ポストフックに頼る代わりに、[grants config](/reference/resource-configs/grants) を使用して、ウェアハウスに権限付与を弾力的に適用します。
- すでに処理されたデータに対して変換が実行されないように、ソースに [source-freshness](/docs/build/sources#source-data-freshness) しきい値を定義します。
- モデルとその上流の依存関係すべてを実行するには、モデルの左側にある `+` 演算子 (`dbt build --select +model_name`) を使用します。モデルとそれに依存する下流のすべてを実行するには、モデルの右側にある `+` 演算子 (`dbt build --select model_name+`) を使用します。
- パッケージまたはディレクトリ内のすべてのモデルを実行するには、`dir_name` を使用します。
- 状態を認識しない CI セットアップでモデルの左側に `@` 演算子を使用して、モデルをテストします。この演算子は、選択したすべての親と子、および新しい CI スキーマではまだ存在しない可能性のある子の親も実行します。
- [--exclude フラグ](/reference/node-selection/exclude)を使用して、選択からモデルのサブセットを削除します。
- 増分モデルを最初から再構築するには、[--full-refresh](/reference/commands/run#refresh-incremental-models) フラグを使用します。
- [seeds](/docs/build/seeds) を使用して、郵便番号から州、マーケティング UTM からキャンペーンなどの手動ルックアップ テーブルを作成します。`dbt seed` は、これらを CSV からウェアハウスに構築し、モデルで `ref` できるようにします。
- [target.name](/docs/build/custom-schemas#an-alternative-pattern-for-generating-schema-names) を使用して、使用している環境に基づいてロジックをピボットします。たとえば、開発中は単一の開発スキーマにビルドし、運用環境では複数のスキーマを使用します。

## 関連ドキュメント

- [クイックスタート ガイド](/guides)
- [dbt Cloud について](/docs/cloud/about-cloud/dbt-cloud-features)
- [クラウドでの開発](/docs/cloud/about-develop-dbt)
