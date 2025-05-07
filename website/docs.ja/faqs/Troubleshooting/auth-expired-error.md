---
title: IDE でクエリを実行しようとすると、「authentication has expired」というエラーが発生します。
description: "`authentication has expired` というエラーが表示されたら、ウェアハウスを再認証します"
sidebar_label: 'IDEで `authentication has expired` というエラーが表示される'
---

dbt Cloud IDE でクエリを実行しようとしたときに `authentication has expired` というエラーが表示される場合は、Snowflake と dbt Cloud 間の [OAuth](/docs/cloud/manage-access/set-up-snowflake-oauth) 接続の有効期限が切れていることを意味します。

この問題を解決するには、2 つのツールを再接続する必要があります。

Snowflake 管理者は、リフレッシュ トークンの有効期間を [構成](/docs/cloud/manage-access/set-up-snowflake-oauth#create-a-security-integration) できます。リフレッシュ トークンの有効期間は最大 90 日間です。

この問題を解決するには、次の手順を実行してください:

1. ナビゲーション メニューから **Profile settings** ページに移動します。
2. **Credentials** に移動し、問題が発生しているプロジェクトをクリックします。
3. **Development credentials** の下にある **Reconnect Snowflake Account**（緑色）ボタンをクリックします。SSOワークフローを使用した再認証の手順が表示されます。

これらの手順を試してもエラーが解消されない場合は、サポートチーム（support@getdbt.com）までお問い合わせください。
