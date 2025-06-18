---
title: "Authentication tokens"
description: "Learn how to authenticate with user tokens and service account tokens "
pagination_next: "docs/dbt-cloud-apis/user-tokens"
pagination_prev: null
---

<div className="grid--2-col">

<Card
    title="個人アクセストークン"
    body="ユーザー トークンと、それを使用して dbt API に対してクエリを実行する方法について学習します。"
    link="/docs/dbt-cloud-apis/user-tokens"
    icon="dbt-bit"/>

<Card
    title="サービスアカウントトークン"
    body="サービス アカウント トークンを使用して、システムレベルの統合のために dbt API で安全に認証する方法を学びます。"
    link="/docs/dbt-cloud-apis/service-tokens"
    icon="dbt-bit"/>

</div>

## APIアクセストークンの種類

**個人アクセストークン:** ユーザーに代わって <Constant name="cloud" /> API にアクセスするための推奨される安全な方法です。PAT はアカウントにスコープが設定されており、よりきめ細かな制御が可能です。

**サービストークン:** サービストークンはサービスアカウントに似ており、<Constant name="cloud" /> アカウントに代わってアクセスを許可するための推奨される方法です。

### どのトークンタイプを使用すべきか

サービスアカウントが必要な本番環境のワークフローでは、サービストークンを広く使用する必要があります。PAT は、開発ワークフロー、またはユーザーコンテキストを必要とする <Constant name="cloud" /> クライアントワークフローにのみ使用してください。以下の例は、個人アクセストークン (PAT) とサービストークンのどちらを使用するべきかを示しています。

* **パートナー統合を <Constant name="cloud" /> に接続する** &mdash; 例としては、[<Constant name="semantic_layer" /> Google Sheets 統合](/docs/cloud-integrations/avail-sl-integrations)、Hightouch、Datafold、独自に作成したカスタムアプリなどが挙げられます。これらのタイプの統合では、PAT ではなくサービストークンを使用する必要があります。サービストークンを使用すると可視性が得られ、統合に必要なスコープのみを設定して最小限の権限を確保できるためです。現在、これらの統合で個人アクセストークンを使用している場合は、サービストークンへの切り替えを強くお勧めします。
* **本番環境の Terraform** - これは本番環境のワークフローであり、ユーザーアカウントではなくサービスアカウントとして機能するため、サービストークンを使用します。
* **<Constant name="cloud_cli" />** - <Constant name="cloud_cli" /> はユーザーのコンテキスト内で機能するため（ユーザーがリクエストを発行し、ユーザーアカウントのコンテキスト内で操作する必要があるため）、PAT を使用します。
* **カスタムスクリプトのテストと Terraform または Postman のステージング** - これは開発ワークフローであり、変更を行うユーザーに限定されるため、PAT を使用することをお勧めします。このスクリプトまたは Terraform を本番環境にプッシュする場合は、代わりにサービストークンを使用します。
