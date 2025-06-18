---
title: "Deploy your metrics"
id: deploy-sl
description: "Deploy the dbt Semantic Layer in dbt by running a job to materialize your metrics."
sidebar_label: "Deploy your metrics"
tags: [Semantic Layer]
pagination_next: "docs/use-dbt-semantic-layer/exports"
---

# Deploy your metrics <Lifecycle status="self_service,managed,managed_plus" />

<!-- The below snippet can be found in the following file locations in the docs code repository) 

https://github.com/dbt-labs/docs.getdbt.com/blob/current/website/snippets/_sl-run-prod-job.md
-->

import RunProdJob from '/snippets.ja/_sl-run-prod-job.md';

<RunProdJob/>

## 次のステップ

ジョブを実行して<Constant name="semantic_layer" />をデプロイしたら、次の手順に従ってください。
- <Constant name="cloud" /> で [<Constant name="semantic_layer" />をセットアップ](/docs/use-dbt-semantic-layer/setup-sl)します。
- Tableau、Google Sheets、Microsoft Excel など、[利用可能な統合](/docs/cloud-integrations/avail-sl-integrations)を確認します。
- [API クエリ構文](/docs/dbt-cloud-apis/sl-jdbc#querying-the-api-for-metric-metadata)を使用して、メトリクスのクエリを開始します。


## 関連ドキュメント

- 宣言型キャッシュを使用して、[クエリパフォーマンスを最適化](/docs/use-dbt-semantic-layer/sl-cache)します。
- [CI でセマンティックノードを検証](/docs/deploy/ci-jobs#semantic-validations-in-ci)して、dbt モデルへのコード変更によってこれらのメトリクスが損なわれないようにします。
- まだお試しでない場合は、お好みの開発ツールで[メトリクスとセマンティックモデルを構築する](/docs/build/build-metrics-intro)方法を学習してください。
