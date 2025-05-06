---
title: "APIの概要"
description: "チーム プランとエンタープライズ プランの dbt アカウントで dbt Cloud API をクエリする方法について説明します。"
id: "overview"
pagination_next: "docs/dbt-cloud-apis/user-tokens"
pagination_prev: null
---

# APIの概要 <Lifecycle status="team,enterprise"/>

_Team_ プランと _Enterprise_ プランのアカウントは、dbt Cloud API にクエリを実行できます。

dbt Cloud は以下の API を提供します。

- [dbt Cloud Administrative API](/docs/dbt-cloud-apis/admin-cloud-api) は、dbt Cloud アカウントの管理に使用できます。この API は手動で呼び出すことも、[dbt Cloud Terraform プロバイダ](https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest) を使用して呼び出すこともできます。
- [dbt Cloud Discovery API](/docs/dbt-cloud-apis/discovery-api) は、dbt プロジェクトの状態と健全性に関するメタデータを取得するために使用できます。
- [dbt Semantic Layer API](/docs/dbt-cloud-apis/sl-api-overview) は、dbt Semantic Layer で定義されたメトリックをクエリするための複数の API オプションを提供します。

Webhook の詳細については、[ジョブ用の Webhook](/docs/deploy/webhooks) を参照してください。

## API へのアクセス方法

dbt Cloud は、[個人アクセストークン](/docs/dbt-cloud-apis/user-tokens) と [サービスアカウントトークン](/docs/dbt-cloud-apis/service-tokens) の 2 種類の API トークンをサポートしています。これらのトークンを使用して、dbt Cloud API へのリクエストを承認できます。
