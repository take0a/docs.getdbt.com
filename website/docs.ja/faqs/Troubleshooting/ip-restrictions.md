---
title: "サービストークンの使用時に403エラー 'Forbidden: Access denied' が表示されます"
description: "すべてのサービストークントラフィックはIP制限の対象となります。403エラーを解決するには、サードパーティ統合のCIDR（ネットワークアドレス）を許可リストに追加してください。"
sidebar_label: 'Service token 403 error: Forbidden: Access denied'
---


すべての [サービストークン](/docs/dbt-cloud-apis/service-tokens) トラフィックは IP 制限の対象となります。

サービストークンの使用時に、次の 403 レスポンスエラーが表示される場合は、IP が許可リストに登録されていないことを示します。この問題を解決するには、サードパーティ統合の CIDR（ネットワークアドレス）を許可リストに追加する必要があります。

以下は 403 レスポンスエラーの例です:

```json
        {
            "status": {
                "code": 403,
                "is_success": False,
                "user_message": ("Forbidden: Access denied"),
                "developer_message": None,
            },
            "data": {
                "account_id": <account_id>,
                "user_id": <user_id>,
                "is_service_token": <boolean describing if it's a service token request>,
                "account_access_denied": True,
            },
        }
```
