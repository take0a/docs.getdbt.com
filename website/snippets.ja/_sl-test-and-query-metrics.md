dbt でメトリクスを操作するには、コマンドを検証または実行するためのツールがいくつかあります。設定に応じて、メトリクスのテストとクエリを実行する方法は次のとおりです。

- [**<Constant name="cloud_ide" /> ユーザー**](#dbt-cloud-ide-users) &mdash; [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) 内で [MetricFlow コマンド](/docs/build/metricflow-commands#metricflow-commands) を直接実行して、メトリクスのクエリ/プレビューを実行します。[**Lineage**] タブでメトリクスを視覚的に表示します。
- [**<Constant name="cloud_cli" /> ユーザー**](#dbt-cloud-cli-users) &mdash; [<Constant name="cloud_cli" />](/docs/cloud/cloud-cli-installation) を使用すると、[MetricFlow コマンド](/docs/build/metricflow-commands#metricflow-commands) を実行して、コマンドラインインターフェースで直接メトリクスのクエリとプレビューを行うことができます。
- **<Constant name="core" /> ユーザー** - コマンド実行には MetricFlow CLI を使用します。このガイドは <Constant name="cloud" /> ユーザーを対象としていますが、<Constant name="core" /> ユーザーの場合は、[MetricFlow コマンド](/docs/build/metricflow-commands#metricflow-commands) ページで MetricFlow CLI の詳細な設定手順を確認できます。<Constant name="semantic_layer" /> を使用するには、[Starter または Enterprise レベルのアカウント](https://www.getdbt.com/) が必要です。

または、DataGrip、DBeaver、RazorSQL などの SQL クライアントツールを使用してコマンドを実行することもできます。

### Studio IDE ユーザー

コマンド名の前に `dbt sl` プレフィックスを付けると、<Constant name="cloud" /> 内でコマンドを実行できます。たとえば、すべてのメトリックを一覧表示するには、`dbt sl list metrics` を実行します。<Constant name="cloud_ide" /> で使用できる MetricFlow コマンドの完全なリストについては、[MetricFlow コマンド](/docs/build/metricflow-commands#metricflow-commandss) ページを参照してください。

メトリックまたはセマンティックモデルの定義にエラーがある場合、エディターの右下にある <Constant name="cloud_ide" /> **ステータスボタン** に **エラー** ステータスが表示されます。このボタンをクリックすると、具体的な問題を確認して解決できます。

確認後、変更内容をプロジェクトにコミットしてマージしてください。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-ide-dag.jpg" title="Validate your metrics using the Lineage tab in the IDE." />

### Cloud CLI ユーザー

このセクションは、<Constant name="cloud_cli" /> ユーザー向けです。MetricFlow コマンドは <Constant name="cloud" /> に統合されているため、<Constant name="cloud_cli" /> をインストールするとすぐに MetricFlow コマンドを実行できます。アカウントが自動的にバージョン管理を行います。

開始するには、以下の手順を参照してください。

1. [<Constant name="cloud_cli" />](/docs/cloud/cloud-cli-installation) をインストールします（まだインストールしていない場合）。次に、dbt プロジェクト ディレクトリに移動します。
2. `dbt parse`、`dbt run`、`dbt compile`、`dbt build` などの dbt コマンドを実行します。実行しないと、「アーティファクトを実行したことを確認してください...」で始まるエラー メッセージが表示されます。
3. MetricFlow はセマンティックグラフを構築し、<Constant name="cloud" /> に `semantic_manifest.json` ファイルを生成します。このファイルは `/target` ディレクトリに保存されます。Jaffle Shop の例を使用する場合は、続行する前に `dbt seed && dbt run` を実行して、必要なデータがデータプラットフォームに存在することを確認してください。

:::tip dbt parse を実行して指標の変更を反映する
指標に変更を加えた場合は、少なくとも `dbt parse` を実行して <Constant name="semantic_layer" /> を更新してください。これにより `semantic_manifest.json` ファイルが更新され、指標のクエリ時に変更が反映されます。`dbt parse` を実行することで、すべてのモデルを再構築する必要がなくなります。
:::

4. `dbt sl --help` を実行して、MetricFlow がインストールされていること、および利用可能なコマンドが表示されることを確認します。
5. `dbt sl query --metrics <metric_name> --group-by <dimension_name>` を実行して、メトリクスとディメンションをクエリします。たとえば、`order_total` と `order_count`（どちらもメトリクス）をクエリし、`order_date`（ディメンション）でグループ化するには、次のコマンドを実行します。

   ```sql
   dbt sl query --metrics order_total,order_count --group-by order_date
   ```
6. メトリック値が期待どおりであることを確認します。メトリックがどのように生成されるかをさらに理解するには、コマンドラインで「--compile」と入力して生成されたSQLを表示できます。
7. メトリック定義を含むコード変更をコミットしてマージします。
