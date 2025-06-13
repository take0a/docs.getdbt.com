## メタデータの生成

<Constant name="explorer" /> は、[Discovery API](/docs/dbt-cloud-apis/discovery-api) によって提供されるメタデータを使用して、[dbt プロジェクトの状態](/docs/dbt-cloud-apis/project-state) に関する詳細を表示します。利用可能なメタデータは、<Constant name="cloud" /> プロジェクトで _production_ または _staging_ として指定した [デプロイメント環境](/docs/deploy/deploy-environments) によって異なります。

<Constant name="explorer" /> を使用すると、Snowflake から外部メタデータを取り込むこともでき、dbt で <Constant name="explorer" /> によって定義されていないテーブル、ビュー、その他のリソースを可視化できます。

## dbt メタデータ

[ハイブリッド プロジェクト セットアップ](/docs/deploy/hybrid-setup) を使用して dbt Core からアーティファクトをアップロードする場合は、[セットアップ手順](/docs/deploy/hybrid-setup#connect-project-in-dbt-cloud) に従って <Constant name="cloud" /> でプロジェクトを接続してください。これにより、<Constant name="explorer" /> がメタデータに正しくアクセスして表示できるようになります。

- すべてのメタデータが <Constant name="explorer" /> で利用可能になるようにするには、本番環境またはステージング環境でジョブの一部として `dbt build` と `dbt docs generate` を実行してください。これらの 2 つのコマンドを実行することで、関連するすべてのメタデータ（系統、テスト結果、ドキュメントなど）が dbt Explorer で利用可能になります。
- <Constant name="explorer" /> は、本番環境またはステージング環境でジョブを実行するたびにメタデータの更新を自動的に取得するため、プロジェクトの最新の結果が常に得られます。これには、デプロイジョブとマージジョブが含まれます。
    - CI ジョブは <Constant name="explorer" /> を更新しません。これは、CI ジョブが本番環境の状態を反映しておらず、必要なメタデータの更新を提供しないためです。
- リソースとそのメタデータを表示するには、プロジェクトでリソースを定義し、本番環境またはステージング環境でジョブを実行する必要があります。
- 結果として得られるメタデータは、ジョブによって実行される [コマンド](/docs/deploy/job-commands) によって異なります。

<Constant name="explorer" /> は、メタデータを更新するジョブが実行されなかった場合、3 か月後に古いメタデータを自動的に削除します。これを回避するには、必要なコマンドを使用して、3 か月よりも頻繁にジョブを実行するようにスケジュールしてください。

| To view in <Constant name="explorer" /> | You must successfully run |
|---------------------|---------------------------|
| All metadata        |  [dbt build](/reference/commands/build), [dbt docs generate](/reference/commands/cmd-docs), and [dbt source freshness](/reference/commands/source#dbt-source-freshness) together as part of the same job in the environment
| Model lineage, details, or results | [dbt run](/reference/commands/run) or [dbt build](/reference/commands/build) on a given model within a job in the environment |
| Columns and statistics for models, sources, and snapshots| [dbt docs generate](/reference/commands/cmd-docs) within [a job](/docs/explore/build-and-view-your-docs) in the environment |
| Test results | [dbt test](/reference/commands/test) or [dbt build](/reference/commands/build) within a job in the environment |
| Source freshness results | [dbt source freshness](/reference/commands/source#dbt-source-freshness) within a job in the environment |
| Snapshot details | [dbt snapshot](/reference/commands/snapshot) or [dbt build](/reference/commands/build) within a job in the environment |
| Seed details | [dbt seed](/reference/commands/seed) or [dbt build](/reference/commands/build) within a job in the environment |

<Constant name="cloud" /> が進化するにつれて、より豊富でタイムリーなメタデータが利用できるようになります。

