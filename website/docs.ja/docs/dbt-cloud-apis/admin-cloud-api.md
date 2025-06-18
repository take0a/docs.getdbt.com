---
title: "dbt Administrative API"
id: "admin-cloud-api"
pagination_next: "docs/dbt-cloud-apis/discovery-api"
---

# dbt Administrative API <Lifecycle status="managed,managed_plus" />

<Constant name="cloud" /> 管理APIは、[EnterpriseプランおよびEnterprise+プラン](https://www.getdbt.com/pricing/)でデフォルトで有効になっています。このAPIは以下の用途に使用できます。

- ジョブ完了後の成果物のダウンロード
- オーケストレーションツールからのジョブ実行の開始
- <Constant name="cloud" /> アカウントの管理
- その他

<Constant name="cloud" /> は現在、Administrative API の v2 と v3 の 2 つのバージョンをサポートしています。一般的には v3 の使用が推奨されますが、すべての v2 ルートが v3 にアップグレードされているわけではありません。現在、この作業を進めています。v3 のドキュメントで見つからない情報がある場合は、v2 エンドポイントの短縮リストをご確認ください。そこに必要な情報が見つかる可能性があります。

管理APIの多くのエンドポイントは、[<Constant name="cloud" /> Terraformプロバイダー](https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest)を介して呼び出すこともできます。Terraformレジストリに組み込まれているドキュメントには、[プロバイダーの使用開始方法に関するガイド](https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest/docs/guides/1_getting_started)と、[設定可能なすべてのTerraformリソースを示すページ](https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest/docs/guides/99_list_resources)が含まれています。

<div className="grid--2-col">

<Card
    title="API v2"
    body="エンドポイントと機能が制限された、旧バージョンのAPIです。v3では利用できない情報が含まれています。"
link="/dbt-cloud/api-v2"
    icon="pencil-paper"/>

<Card
    title="API v3"
    body="新しいエンドポイントと機能を備えた最新の API バージョン。"
link="/dbt-cloud/api-v3"
    icon="pencil-paper"/>

<div className="card-container">
 <Card
    title="dbt Terraform provider"
    link="https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest"
    body="dbt Labs によって管理されている Terraform プロバイダー。dbt アカウントの管理に使用できます。"
    icon="pencil-paper"/>
    <a href="https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest"
    className="external-link"
    target="_blank"
    rel="noopener noreferrer">
    <Icon name='fa-external-link' />
  </a>
</div>

</div>
