---
title: "他のオーケストレーションツールとの統合"
id: "deployment-tools"
sidebar_label: "Integrate with other tools"
pagination_next: null
---

[dbt Cloud](/docs/deploy/jobs)に加えて、このページで説明されているようなツールを活用して、dbtジョブをスケジュールして実行する他の方法もご確認ください。

これらのツールをビルドしてインストールすることで、データワークフローを自動化し、dbtジョブ（dbt Cloudでホストされているジョブを含む）をトリガーし、手間のかからない操作性を実現し、時間を節約し、効率性を向上させることができます。

## Airflow

組織で [Airflow](https://airflow.apache.org/) を使用している場合、次のようなさまざまな方法で dbt ジョブを実行できます:

<Tabs>

<TabItem value="airflowcloud" label="dbt Cloud">

dbt Cloud ジョブをオーケストレーションするために、[dbt Cloud Provider](https://airflow.apache.org/docs/apache-airflow-providers-dbt-cloud/stable/index.html) をインストールします。このパッケージには、dbt Cloud 内でさまざまなアクションを実行するための複数のフック、オペレーター、センサーが含まれています。

<Lightbox src="/img/docs/running-a-dbt-project/airflow_dbt_connector.png" title="Airflow DAG using DbtCloudRunJobOperator"/>
<Lightbox src="/img/docs/running-a-dbt-project/dbt_cloud_airflow_trigger.png" title="dbt Cloud job triggered by Airflow"/>

</TabItem>

<TabItem value="airflowcore" label="dbt Core">

[BashOperator](https://registry.astronomer.io/providers/apache-airflow/modules/bashoperator) を介して dbt Core ジョブを呼び出す。この場合、Airflow と dbt 間の依存関係の競合による問題を回避するため、dbt を仮想環境にインストールしてください。

</TabItem>
</Tabs>

これら 2 つの方法の詳細 (実装例を含む) については、[このガイド](https://docs.astronomer.io/learn/airflow-dbt-cloud) を参照してください。

## 自動化サーバー

自動化サーバー（CodeDeploy、GitLab CI/CD（[動画](https://youtu.be/-XBIIY2pFpc?t=1301)）、Bamboo、Jenkins など）を使用して、dbt の bash コマンドをスケジュールできます。また、コマンドラインへのログ出力を表示したり、Git リポジトリと統合したりするための UI も提供しています。

## Azure Data Factory

dbt Cloud と [Azure Data Factory](https://learn.microsoft.com/en-us/azure/data-factory/) (ADF) を統合することで、データの取り込みから変換まで、スムーズなデータ処理が可能になります。ADF の [dbt API](/docs/dbt-cloud-apis/overview) を使用することで、取り込みジョブの完了時に dbt Cloud ジョブをシームレスにトリガーできます。

次のビデオでは、Azure Data Factory の API を介して dbt Cloud ジョブをトリガーする方法の詳細な概要を説明しています。

<LoomVideo id="8dcc1d22a0bf43a1b89ecc6f6b6d0b18" /> 


dbt API を使用して ADF 経由で dbt Cloud のジョブをトリガーするには、次の手順を実行します。

1. dbt Cloud で、日次本番ジョブのジョブ設定に移動し、[**トリガー**] セクションでスケジュールされた実行をオフにします。
2. ADF で dbt Cloud ジョブをトリガーするためのパイプラインを作成します。
3. パイプラインの最初のステップとして Web 呼び出しを使用して、ADF のキー コンテナーから dbt Cloud サービス トークンを安全に取得します。
4. パイプラインで、dbt Cloud アカウント ID、ジョブ ID、キー コンテナーの名前、サービス トークンを含むシークレットなどのパラメータを設定します。
* dbt Cloud ジョブとアカウント ID は URL に記載されています。たとえば、URL が `https://YOUR_ACCESS_URL/deploy/88888/projects/678910/jobs/123456` の場合、アカウント ID は 88888、ジョブ ID は 123456 です。
5. ADF でパイプラインをトリガーして dbt Cloud ジョブを開始し、ADF で dbt Cloud ジョブのステータスを監視します。
6. dbt Cloud で、ジョブのステータスと、dbt Cloud でジョブがどのようにトリガーされたかを確認できます。

## Cron

cronはbashコマンドをスケジュールするのに有効な手段です。しかし、ジョブをスケジュールする簡単な方法のように見えるかもしれませんが、本番環境へのデプロイに関連する追加機能をすべて処理するコードを書く必要があるため、ここで挙げた他の方法と比べて、この方法はより複雑になることが多いです。

## Dagster

組織で [Dagster](https://dagster.io/) をご利用の場合は、[dagster_dbt](https://docs.dagster.io/_apidocs/libraries/dagster-dbt) ライブラリを使用して、dbt コマンドをパイプラインに統合できます。このライブラリは、dbt Cloud または dbt Core を介した dbt の実行をサポートしています。Dagster から dbt を実行すると、dbt 実行に関するメタデータが自動的に集約されます。詳細については、[サンプルパイプライン](https://dagster.io/blog/dagster-dbt) を参照してください。

## Databricks ワークフロー

Databricks ワークフローを使用して dbt Cloud ジョブ API を呼び出すと、他の ETL プロセスとの統合、dbt Cloud ジョブ機能の活用、関心の分離、カスタム条件またはロジックに基づくカスタムジョブのトリガーなど、さまざまなメリットが得られます。これらのメリットにより、モジュール性の向上、デバッグの効率化、dbt Cloud ジョブのスケジュール設定の柔軟性が向上します。

詳細については、[Databricks ワークフローと dbt Cloud ジョブ](/guides/how-to-use-databricks-workflows-to-run-dbt-cloud-jobs) に関するガイドを参照してください。

## Kestra

組織で [Kestra](http://kestra.io/) を使用している場合は、[dbt プラグイン](https://kestra.io/plugins/plugin-dbt) を利用して dbt Cloud ジョブと dbt Core ジョブをオーケストレーションできます。Kestra のユーザー インターフェース (UI) には [ブループリント](https://kestra.io/docs/user-interface-guide/blueprints) が組み込まれており、すぐに使用できるワークフローが提供されています。左側のナビゲーション メニューの [ブループリント](https://kestra.io/docs/user-interface-guide/blueprints) ページで [dbt タグを選択](https://demo.kestra.io/ui/blueprints/community?selectedTag=36) すると、データ パイプラインの一部として dbt Core コマンドと dbt Cloud ジョブをスケジュールするいくつかの例が表示されます。スケジュールされたワークフローまたはアドホック ワークフローの実行ごとに、Kestra UI の [出力] タブで、すべての dbt ビルド成果物をダウンロードしてプレビューできます。ガントチャートとトポロジビューでは、メタデータも表示され、dbtモデルとテストの依存関係と実行時間を視覚化できます。dbt Cloudタスクには、Kestraとdbt Cloud UI間を簡単に移動するための便利なリンクが用意されています。

## Orchestra

組織で [Orchestra](https://getorchestra.io) を使用している場合は、dbt Cloud API を使用して dbt ジョブをトリガーできます。dbt Cloud アカウントから API トークンを作成し、これを使用して [Orchestra ポータル](https://app.getorchestra.io) で Orchestra を認証します。詳細については、[dbt Cloud の Orchestra ドキュメント](https://orchestra-1.gitbook.io/orchestra-portal/integrations/transformation/dbt-cloud) を参照してください。

Orchestra は実行からメタデータを自動的に収集するため、dbt ジョブを他のデータスタックのコンテキストで表示できます。

以下は、Orchestra によってトリガーされたジョブの dbt Cloud での実行詳細の例です。

<Lightbox src="/img/docs/running-a-dbt-project/dbt_cloud_orchestra_trigger.png" title="Example of Orchestra triggering a dbt job"/>

以下は、Orchestra で dbt ジョブの系統を表示する例です:

<Lightbox src="/img/docs/running-a-dbt-project/orchestra_lineage_dbt_cloud.png" title="Example of a lineage view for dbt jobs in Orchestra"/>


## Prefect

組織で[Prefect](https://www.prefect.io/)をご利用の場合、ジョブの実行方法は、dbtのバージョンと、dbt Cloudジョブとdbt Coreジョブのどちらをオーケストレーションするかによって異なります。以下のオプションをご参照ください:

<Lightbox src="/img/docs/running-a-dbt-project/prefect_dag_dbt_cloud.jpg" width="75%" title="Prefect DAG using a dbt Cloud job run flow"/> 


### Prefect 2

<Tabs>

<TabItem value="prefect2cloud" label="dbt Cloud">

- [trigger_dbt_cloud_job_run_and_wait_for_completion](https://prefecthq.github.io/prefect-dbt/cloud/jobs/#prefect_dbt.cloud.jobs.trigger_dbt_cloud_job_run_and_wait_for_completion) フローを使用します。
- ジョブの実行中に、[Prefect ユーザーインターフェース (UI)](https://docs.prefect.io/ui/overview/) を介して dbt をポーリングし、ジョブが失敗なく完了したかどうかを確認できます。

<Lightbox src="/img/docs/running-a-dbt-project/dbt_cloud_job_prefect.jpg" title="dbt Cloud job triggered by Prefect"/> 

</TabItem>

<TabItem value="prefect2core" label="dbt Core">

- [trigger_dbt_cli_command](https://prefecthq.github.io/prefect-dbt/cli/commands/#prefect_dbt.cli.commands.trigger_dbt_cli_command) タスクを使用します。
- これらの方法の詳細については、[prefect-dbt ドキュメント](https://prefecthq.github.io/prefect-dbt/) を参照してください。

</TabItem>
</Tabs>


### Prefect 1

<Tabs>

<TabItem value="prefect1cloud" label="dbt Cloud">

- [DbtCloudRunJob](https://docs.prefect.io/api/latest/tasks/dbt.html#dbtcloudrunjob) タスクを使用して、dbt Cloud ジョブをトリガーします。
- このタスクを実行すると、Prefect UI で表示可能なマークダウン アーティファクトが生成されます。
- このアーティファクトには、ジョブ実行の結果として生成された dbt アーティファクトへのリンクが含まれます。

</TabItem>

<TabItem value="prefect1core" label="dbt Core">

- [DbtShellTask​​](https://docs.prefect.io/api/latest/tasks/dbt.html#dbtshelltask) を使用して、dbt 実行のスケジュール設定、実行、監視を行います。
- サポートされている [ShellTask​​](https://docs.prefect.io/api/latest/tasks/shell.html#shelltask) を使用して、シェル経由で dbt コマンドを実行します。


</TabItem>
</Tabs>


## 関連ドキュメント

- [dbt Cloud のプランと料金](https://www.getdbt.com/pricing/)
- [クイックスタートガイド](/guides)
- [ジョブ用 Webhook](/docs/deploy/webhooks)
- [オーケストレーションガイド](https://docs.getdbt.com/guides/orchestration)
- [本番環境デプロイメント用のコマンド](https://discourse.getdbt.com/t/what-are-the-dbt-commands-you-run-in-your-production-deployment-of-dbt/366)
