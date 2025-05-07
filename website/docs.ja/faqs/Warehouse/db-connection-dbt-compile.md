---
title: dbt compile にデータ プラットフォーム接続が必要な理由
description: "`dbt compile` は、ウェアハウスの現在の状態に応じて作業を行うため、データ プラットフォーム接続が必要です。"
sidebar_label: "dbt compile にデータ プラットフォーム接続が必要な理由"
id: db-connection-dbt-compile
---

`dbt compile` では、プロジェクト内のすべてのモデルの SQL を準備するために必要な情報 (イントロスペクト クエリからの情報を含む) を収集するために、データ プラットフォーム接続が必要です。

### dbt compile

[`dbt compile` コマンド](/reference/commands/compile) は、`source`、`model`、`test`、`analysis` ファイルから実行可能な SQL を生成します。`dbt compile` は、モデルのコンパイル済み SQL を既存のテーブルにマテリアライズしない点を除けば、`dbt run` と似ています。マテリアライズの時点までは、`dbt compile` と `dbt run` はどちらもデータプラットフォーム接続を必要とし、クエリを実行し、[`execute` 変数](/reference/dbt-jinja-functions/execute) が `True` に設定されている点で似ています。

ただし、考慮すべき点がいくつかあります。

- `dbt run` の前に `dbt compile` を実行する必要はありません。
- dbt では、`compile` は `parse` を意味しません。これは、`parse` が記述した `YAML` や設定されたタグなどを検証するためです。

### イントロスペクティブクエリ

多くのモデルに対してコンパイル済み SQL を生成するために、dbt はデータプラットフォームに対してイントロスペクティブクエリ（データを取得して処理するために SQL を実行する必要があるクエリ）を実行する必要があります。

これらのイントロスペクティブクエリには、次のものが含まれます。

- リレーションキャッシュへのデータ入力。詳細については、[新しいマテリアライゼーションの作成](/guides/create-new-materializations)ガイドを参照してください。キャッシュにより、メタデータチェック（[増分モデル](/docs/build/incremental-models)がデータプラットフォームに既に存在するかどうかの確認など）が高速化されます。
- SQL のテンプレート化に使用している `run_query` や `dbt_utils.get_column_values` などの[マクロ](/docs/build/jinja-macros#macros) の解決。これは、dbt がモデル SQL のコンパイル中にこれらのクエリを実行する必要があるためです。

データ プラットフォーム接続がないと、dbt はこれらのイントロスペクティブ クエリを実行できず、dbt ワークフローの次のステップに必要なコンパイル済み SQL を生成できません。インターネット接続やデータ プラットフォーム接続がなくても、プロジェクトを [`parse`](/reference/commands/parse) し、プロジェクト内の [`list`](/reference/commands/list) リソースを使用できます。プロジェクトを解析するだけで [マニフェスト](/reference/artifacts/manifest-json) を作成できますが、書き出されたマニフェストにはコンパイル済み SQL が含まれないことに注意してください。

プロジェクトを構成するには、[接続プロファイル](/docs/core/connect-data-platform/connection-profiles) (CLI を使用する場合は `profiles.yml`) が必要です。プロジェクトの構成はこのファイルの内容に依存するため、このファイルが必要になります。たとえば、条件付き構成に [`{{target}}`](/reference/dbt-jinja-functions/target) を使用したり、実行しているプラ​​ットフォームを把握して適切な SQL フレーバーを選択したりする必要がある場合があります。



