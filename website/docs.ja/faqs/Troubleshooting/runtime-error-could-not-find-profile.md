---
title: ランタイム エラー「Could not find profile named 'user'」というエラーが発生します。
description: "プロフィール設定で資格情報を再認証する"
sidebar_label: 'IDEで「Could not find profile named user"error」'
id: runtime-error-could-not-find-profile

---

以下のエラー メッセージが表示されて IDE にアクセスできない場合は、以下の手順に従って問題を解決できるよう最善を尽くします。

```shell
Running with dbt=1.9.0
Encountered an error while reading the project:
  ERROR: Runtime Error
  Could not find profile named 'user'
Runtime Error
  Could not run dbt'
```

通常、このエラーは、資格情報または認証情報が不足しているか古くなっていることが原因で発生します。ご安心ください。いくつかの回避策をお試しください。

**IDE の場合:**
IDE でこのエラーが発生する場合は、開発用の資格情報が設定されているプロファイル設定に移動してください。設定画面が表示されたら、資格情報を再入力するか、再認証することで、このエラーメッセージを回避することができます。

**ジョブの場合:**
ジョブでこのエラーが発生する場合は、ジョブが設定されているデプロイメント環境に何らかの変更を加えたものの、変更を保存する際にデプロイメント用の資格情報を再入力しなかった可能性があります。この問題を解決するには、デプロイメント環境設定に戻り、資格情報（秘密鍵/秘密鍵のパスフレーズ、またはユーザー名とパスワード）を再入力して、新しいジョブ実行を開始する必要があります。

上記の手順を試してもこの現象が継続する場合は、support@getdbt.com のサポート チームまでご連絡ください。喜んでお手伝いいたします。
