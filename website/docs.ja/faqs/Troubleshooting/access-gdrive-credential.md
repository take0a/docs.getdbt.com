---
title: Google ドライブからクエリを実行しようとすると、「ドライブの認証情報の取得中にアクセスが拒否されました」というエラーが表示されます。
description: "BigQuery サービス アカウントへのアクセスを許可する"
sidebar_label: 'Google ドライブからクエリを実行しようとしたときにエラーが発生しました'
id: access-gdrive-credential

---

IDE で Google Drive ドキュメントからデータセットをクエリしようとしたときに以下のエラーが表示される場合、IDE で以下のエラー メッセージが表示されるため、以下の手順で問題を解決できるよう最善を尽くします。

```
Access denied: BigQuery BigQuery: Permission denied while getting Drive credentials
```

通常、このエラーは、BigQuery サービス アカウントに特定の Google ドライブ ドキュメントへのアクセスを許可していないことを示します。このエラーが表示される場合は、dbt Cloud で BigQuery 接続に使用しているサービス アカウント（[こちら](/docs/cloud/connect-data-platform/connect-bigquery) に記載されているクライアントのメール アドレス）に、Google ドライブまたは Google スプレッドシートへのアクセス権限を付与してみてください。この操作は、Google ドキュメント内で直接実行し、**Share** ボタンをクリックしてクライアントのメール アドレスを入力してください。

OAuth の使用時にこのエラーが発生し、Google スプレッドシートへのアクセスを検証済みの場合は、gcloud に Google ドライブへのアクセス権限を付与する必要がある可能性があります。

```
gcloud auth application-default login --disable-quota-project
```

詳細については、[gcloud auth application-default のドキュメント](https://cloud.google.com/sdk/gcloud/reference/auth/application-default/login) をご覧ください。

上記の手順を試してもこの動作が続く場合は、以下のコマンドを使用して Google Cloud にログインし、Google ドライブへのアクセスを有効にしてください。このコマンドは、多くの Google Cloud ライブラリが API 呼び出しの認証に使用するアプリケーション デフォルト認証情報 (ADC) ファイルも更新します。

```
gcloud auth login --enable-gdrive-access --update-adc
```

詳細については、[gcloud auth login ドキュメント](https://cloud.google.com/sdk/gcloud/reference/auth/login#--enable-gdrive-access) をご覧ください。

上記の手順を試しても問題が解決しない場合は、サポートチーム（support@getdbt.com）までお問い合わせください。喜んでお手伝いいたします。
