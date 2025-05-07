---
title: BigQuery で適切な権限を設定するにはどうすればよいですか?
description: "サービス アカウントを使用して BigQuery の権限を設定する"
sidebar_label: 'BigQuery での権限の設定'
id: bq-impersonate-service-account-setup

---

この機能を使用するには、まず、権限を借用するサービス アカウントを作成します。次に、このサービス アカウントを権限借用できるようにするユーザーに、サービス アカウント リソースに対する `roles/iam.serviceAccountTokenCreator` ロールを付与します。さらに、サービス アカウント自体にも同じロールを付与する必要があります。これにより、サービス アカウントは自身を識別するための短命トークンを作成できるようになり、人間のユーザー（または他のサービス アカウント）も同様のトークンを作成できるようになります。このシナリオの詳細については、
[こちら](https://cloud.google.com/iam/docs/understanding-service-accounts#directly_impersonating_a_service_account) をご覧ください。

適切な権限を付与したら、[IAM Service Account Credentials API](https://console.cloud.google.com/apis/library/iamcredentials.googleapis.com) を有効にする必要があります。
API の有効化とロールの付与は結果整合性のある操作であり、完了までに最大 7 分かかりますが、通常は 60 秒以内に完全に反映されます。数分待ってから、BigQuery プロファイル設定に `impersonate_service_account` オプションを追加してください。