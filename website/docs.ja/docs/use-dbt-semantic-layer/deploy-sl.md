---
title: "メトリクスを展開する"
id: deploy-sl
description: "メトリクスを具体化するジョブを実行して、dbt Cloud に dbt セマンティック レイヤーをデプロイします。"
sidebar_label: "Deploy your metrics"
tags: [Semantic Layer]
pagination_next: "docs/use-dbt-semantic-layer/exports"
---

<!-- The below snippet can be found in the following file locations in the docs code repository) 

https://github.com/dbt-labs/docs.getdbt.com/blob/current/website/snippets/_sl-run-prod-job.md
-->

import RunProdJob from '/snippets/_sl-run-prod-job.md';

<RunProdJob/>

## 次のステップ

ジョブを実行してセマンティックレイヤーをデプロイしたら、次の手順に従ってください。
- dbt Cloud で [セマンティックレイヤーをセットアップ](/docs/use-dbt-semantic-layer/setup-sl)します。
- Tableau、Google Sheets、Microsoft Excel など、[利用可能な統合](/docs/cloud-integrations/avail-sl-integrations)を確認します。
- [API クエリ構文](/docs/dbt-cloud-apis/sl-jdbc#querying-the-api-for-metric-metadata)を使用して、メトリクスのクエリを開始します。


## 関連ドキュメント

- 宣言型キャッシュを使用して、[クエリパフォーマンスを最適化](/docs/use-dbt-semantic-layer/sl-cache)します。
- [CI でセマンティックノードを検証](/docs/deploy/ci-jobs#semantic-validations-in-ci)して、dbt モデルへのコード変更によってこれらのメトリクスが損なわれないようにします。
- まだお試しでない場合は、お好みの開発ツールで[メトリクスとセマンティックモデルを構築する](/docs/build/build-metrics-intro)方法を学習してください。
