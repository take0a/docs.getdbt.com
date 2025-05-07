---
title: IDE で NoneType オブジェクトに属性がないというエラーが発生します。
description: "SSHキーをデータウェアハウスにコピーする"
sidebar_label: 'IDEのNoneTypeエラー'
id: nonetype-ide-error

---

以下のエラー メッセージが表示されて IDE にアクセスできない場合は、以下の手順に従って問題を解決できるよう最善を尽くします。

```shell
NoneType object has no attribute 
enumerate_fields'
```

通常、このエラーは、[SSH トンネル](https://docs.getdbt.com/docs/dbt-cloud/cloud-configuring-dbt-cloud/connecting-your-database#connecting-via-an-ssh-tunnel) 経由でデータベースに接続しようとしたことを示しています。このエラーが表示される場合は、以下の項目が指定されているか再度ご確認ください。

- ホスト名
- ユーザー名
- 要塞サーバーのポート番号

上記の手順を試しても問題が解決しない場合は、サポートチーム（support@getdbt.com）までお問い合わせください。喜んでお手伝いいたします。
