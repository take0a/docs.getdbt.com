---
title: dbt Cloud ジョブで 'Could not parse dbt_project.yml' というエラーが発生する
description: "dbt Cloud で 'Could not parse dbt_project.yml' というエラーが表示されましたか? このエラーは通常、dbt_project.yml ファイルのタブ インデントによって発生します。"
sidebar_label: 'dbt Cloud で dbt_project.yml を解析できませんでした'
---

dbt Cloud ジョブの実行中または開発中に `Could not parse dbt_project.yml: while scanning for...` というエラーメッセージが表示される場合、通常はいくつかの原因が考えられます。

- YAML ファイルの解析エラー（タブインデントや Unicode 文字など）
- `dbt_project.yml` ファイルにフィールドが欠落しているか、フォーマットが正しくありません。
- `dbt_project.yml` ファイルが dbt プロジェクト リポジトリに存在しません。

この問題を解決するには、次の点を検討してください。
- オンライン YAML パーサーまたはバリデータを使用して、YAML ファイルに解析エラーがないか確認します。既知の解析エラーには、フィールドの欠落、フォーマットが正しくない、タブインデントなどがあります。
- または、`dbt_project.yml` ファイルが存在することを確認してください。

問題を特定したら、エラーを修正して dbt Cloud ジョブを再実行できます。
