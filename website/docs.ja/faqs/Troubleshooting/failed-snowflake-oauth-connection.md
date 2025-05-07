---
title: Snowflake に接続する際に `Failed to connect to DB` というエラーが表示される
description: "エラーが表示されたらOAuthセキュリティ統合を編集してください"
sidebar_label: '`Failed to connect to database` というエラーが表示される'
---

1. 次のエラーが表示された場合:

   ```text
   Failed to connect to DB: xxxxxxx.snowflakecomputing.com:443. The role requested in the connection, or the default role if none was requested in the connection ('xxxxx'), is not listed in the Access Token or was filtered. 
   Please specify another role, or contact your OAuth Authorization server administrator.
   ```

2. OAuth セキュリティ統合を編集し、このスコープ マッピング属性を明示的に指定します:

   ```sql
   ALTER INTEGRATION <my_int_name> SET EXTERNAL_OAUTH_SCOPE_MAPPING_ATTRIBUTE = 'scp';
   ```

このエラーの詳細については、[Snowflake のドキュメント](https://community.snowflake.com/s/article/external-custom-oauth-error-the-role-requested-in-the-connection-is-not-listed-in-the-access-token) をご覧ください。

----

1. 次のエラーが表示された場合:

   ```text
   Failed to connect to DB: xxxxxxx.snowflakecomputing.com:443. Incorrect username or password was specified.
   ```

   * **一意のメールアドレス** &mdash; Snowflake の各ユーザーは、一意のメールアドレスを持っている必要があります。複数のユーザー（人間のユーザーとサービスアカウントなど）が、`alice@acme.com` などの同じメールアドレスを使用して Snowflake に認証することはできません。
   * **メールアドレスを ID プロバイダーと一致させる** &mdash; Snowflake ユーザーのメールアドレスは、ID プロバイダー (IdP) での認証に使用するメールアドレスと完全に一致する必要があります。たとえば、Snowflake ユーザーのメールアドレスが `alice@acme.com` であるのに、Entra または Okta に `alice_adm@acme.com` を使用してログインすると、この不一致によってエラーが発生する可能性があります。
