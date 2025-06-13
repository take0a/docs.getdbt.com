## Canvas の前提条件
<Constant name="visual_editor" /> を使用する前に、次の要件を満たす必要があります。
- [<Constant name="cloud" /> Enterprise または Enterprise+](https://www.getdbt.com/pricing) アカウントが必要です。
- 開発者認証情報が設定された [開発者ライセンス](/docs/cloud/manage-access/seats-and-users) が必要です。
- 次のいずれかのアダプタを使用している必要があります。
    - BigQuery
    - Databricks
    - Redshift
    - Snowflake
    - Trino
    - リストされていないアダプタでも <Constant name="visual_editor" /> にアクセスできますが、現時点では一部の機能が利用できない場合があります。
- <Constant name="git" /> プロバイダーとして [GitHub](/docs/cloud/git/connect-github) または [GitLab](/docs/cloud/git/connect-gitlab) を使用している必要があります。
- 既存の <Constant name="cloud" /> プロジェクトが作成されており、本番環境での実行が完了していること。
- 開発環境がサポートされている [リリース トラック](/docs/dbt-versions/cloud-release-tracks) であること、および継続的なアップデートを受信できることを確認してください。
- <Constant name="visual_editor" /> で `run` を実行できるデータを含む [ステージング環境](/docs/deploy/deploy-environments#staging-environment) への読み取り専用アクセス権を持っていること。<Constant name="visual_editor" /> ユーザー グループに必要なアクセス権限をカスタマイズするには、詳細については [環境レベルの権限の設定](/docs/cloud/manage-access/environment-permissions-setup) を参照してください。
- AI 活用機能のトグルが有効になっていること（[<Constant name="copilot" /> 統合](/docs/cloud/dbt-copilot) 用）。