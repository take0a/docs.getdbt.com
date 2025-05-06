---
title: "dbtをデプロイする"
id: "deployments"
sidebar: "dbt Cloud の機能を使用して、本番環境で dbt ジョブをシームレスに実行します。"
hide_table_of_contents: true
tags: ["scheduler"]
pagination_next: "docs/deploy/job-scheduler"
pagination_prev: null
---

<IntroText>

dbt Cloud の機能を活用することで、本番環境またはステージング環境で dbt ジョブをシームレスに実行できます。コマンドラインから手動で dbt コマンドを実行する代わりに、[dbt Cloud のアプリ内スケジューリング](/docs/deploy/job-scheduler) を活用して、dbt の実行方法とタイミングを自動化できます。

</IntroText>

dbt Cloudは、dbtプロジェクトを本番環境で実行するための最も簡単かつ信頼性の高い方法を提供します。開発段階から本番環境へと高品質なコードを容易に移行し、ビジネスインテリジェンスツールやエンドユーザーがビジネス上の意思決定に活用できる最新のデータ資産を構築できます。 dbt Cloud を使用した <Term id="deploying">デプロイ</Term> により、次のことが可能になります。
- 本番環境データをタイムリーに最新の状態に保つ
- CI および本番環境パイプラインの効率性を確保する
- デプロイメント環境における障害の根本原因を特定する
- 本番環境で高品質なコードとデータを維持する
- デプロイメント ジョブ、モデル、テストの [健全性](/docs/collaborate/data-tile) を可視化する
- [エクスポート](/docs/use-dbt-semantic-layer/exports) を使用して、データ プラットフォームに [保存済みクエリ](/docs/build/saved-queries) を記述し、信頼性の高い高速なメトリクス レポートを作成する
- 下流のエクスポージャーを [視覚化](/docs/cloud-integrations/downstream-exposures-tableau) および [オーケストレーション](/docs/cloud-integrations/orchestrate-exposures) することで、下流のツールでモデルがどのように使用されているかを把握し、スケジュールされた dbt ジョブ中に基盤となるデータソースをプロアクティブに更新する。 <Lifecycle status="enterprise"/>
- [dbt Cloud の Git リポジトリ キャッシュ](/docs/cloud/account-settings#git-repository-caching) を使用して、サードパーティの障害から保護し、ジョブ実行の信頼性を向上させます。<Lifecycle status="enterprise" />

続行する前に、dbt の [デプロイメント環境](/docs/deploy/deploy-environments) に対するアプローチを理解していることを確認してください。

dbt Cloud の機能を活用して、チームがタイムリーかつ高品質な本番環境データをより簡単に提供できるようにする方法を学びましょう。

## Deploy with dbt

<div className="grid--3-col">

<Card
    title="Job scheduler"
    body="ジョブ スケジューラは、dbt Cloud でジョブを実行するためのバックボーンであり、継続的インテグレーション環境と実稼働環境の両方でデータ パイプラインの構築にパワーとシンプルさをもたらします。"
    link="/docs/deploy/job-scheduler"
    icon="dbt-bit"/>

<Card
    title="Deploy jobs"
    body="ジョブ スケジューラが実行するジョブを作成し、スケジュールします。<br /><br />スケジュールに従って、API によって、または別のジョブの完了後に実行されます。"
    link="/docs/deploy/deploy-jobs"
    icon="dbt-bit"/>

<Card
    title="Continuous integration"
    body="CI チェックを設定すると、PR を開いて新しいコミットを dbt リポジトリにプッシュするときに、変更されたコードをステージング環境でビルドしてテストできるようになります。"
    link="/docs/deploy/continuous-integration"
    icon="dbt-bit"/>

<Card
    title="Continuous deployment"
    body="プル リクエストが Git リポジトリにマージされるときに、最新のコード変更が常に本番環境に反映されるように、マージ ジョブを設定します。"
    link="/docs/deploy/continuous-deployment"
    icon="dbt-bit"/>

<Card
    title="Job commands"
    body="dbt ジョブを実行するときに実行する dbt コマンドを構成します。"
    link="/docs/deploy/job-commands"
    icon="dbt-bit"/>

</div> <br />

## Monitor jobs and alerts

<div className="grid--3-col">

<Card
    title="Visualize and orchestrate exposures"
    body="dbt Cloud を使用してダッシュボードからダウンストリーム エクスポージャーを自動的に生成し、スケジュールされた dbt ジョブ中に基礎となるデータ ソースをプロアクティブに更新する方法を学習します。"
    link="/docs/deploy/orchestrate-exposures"
    icon="dbt-bit"/>

<Card
    title="Artifacts"
    body="dbt Cloud を使用してダッシュボードからダウンストリーム エクスポージャーを自動的に生成し、スケジュールされた dbt ジョブ中に基礎となるデータ ソースをプロアクティブに更新する方法を学習します。"
    link="/docs/deploy/artifacts"
    icon="dbt-bit"/>

<Card
    title="Job notifications"
    body="ジョブの実行が成功、失敗、またはキャンセルされたときにメールまたは Slack チャネルの通知を受信して​​、迅速に対応し、必要に応じて修復を開始できます。"
    link="/docs/deploy/job-notifications"
    icon="dbt-bit"/>

<Card
    title="Model notifications"
    body="ジョブの実行中にモデルやテストで発生した問題に関する電子メール通知をリアルタイムで受信します。"
    link="/docs/deploy/model-notifications"
    icon="dbt-bit"/>

<Card
    title="Run visibility"
    body="実行履歴とモデルタイミングダッシュボードを表示して、スケジュールされたジョブの改善点を特定するのに役立ちます。"
    link="/docs/deploy/run-visibility"
    icon="dbt-bit"/>

<Card
    title="Retry jobs"
    body="エラーの発生したジョブを、開始または障害発生時点から再実行します。"
    link="/docs/deploy/retry-jobs"
    icon="dbt-bit"/>

<Card
    title="Source freshness"
    body="スナップショットを有効にしてデータソースの鮮度をキャプチャし、スナップショットの取得頻度を設定できます。これにより、ソースデータの鮮度がSLAを満たしているかどうかを判断できます。"
    link="/docs/deploy/source-freshness"
    icon="dbt-bit"/>

<Card
    title="Webhooks"
    body="アウトバウンド Webhook を作成して、dbt ジョブのステータスに関するイベントを組織内の他のシステムに送信します。"
    link="/docs/deploy/webhooks"
    icon="dbt-bit"/>

</div> <br />


<!--
<a href="https://docs.getdbt.com/docs/deploy/dbt-cloud-job" target="_blank" class="pagination-nav__label nav-create-account button button--primary">Try deploying with dbt Cloud</a> 

<DocCarousel slidesPerView={1}>

<Lightbox src="/img/docs/dbt-cloud/deployment/deploy-scheduler.jpg" width="98%" title="An overview of a dbt Cloud job run which contains Run Summary, Job Trigger, Run Duration, and more."/>

<Lightbox src="/img/docs/dbt-cloud/deployment/run-history.jpg" width="95%" title="Run History dashboard allows you to monitor the health of your dbt project and displays jobs, job status, environment, timing, and more."/>


<Lightbox src="/img/docs/dbt-cloud/deployment/access-logs.gif" width="85%" title="Access logs for run steps" />

<Lightbox src ="/img/docs/dbt-cloud/using-dbt-cloud/job-commands.gif" width="95%" title="Setting up a job and configuring checkbox and dbt commands"/>

</DocCarousel>

## Run dbt in production

If you want to run dbt jobs on a schedule, you can use tools such as dbt Cloud, Airflow, Prefect, Dagster, automation server, or Cron.-->

## 関連ドキュメント

- [エクスポートを使用して保存済みクエリをマテリアライズする](/docs/use-dbt-semantic-layer/exports)
- [他のオーケストレーションツールと統合する](/docs/deploy/deployment-tools)
