---
title: GitLabでCIジョブをトリガーできない
description: "CIジョブをトリガーできません"
sidebar_label: 'CIジョブをトリガーできません'
id: gitlab-webhook
---

dbt Cloud を GitLab リポジトリに接続すると、GitLab はバックグラウンドで自動的に Webhook を登録します。登録された Webhook はリポジトリ設定で確認できます。この Webhook は、リポジトリへのプッシュ時に [CI ジョブ](/docs/deploy/ci-jobs) をトリガーするためにも使用されます。

CI ジョブをトリガーできない場合は、通常、Webhook の登録が欠落しているか、正しくないことを示しています。

この問題を解決するには、GitLab のリポジトリ設定に移動し、GitLab --> **Settings** --> **Webhooks** に移動して Webhook の登録を確認します。

確認事項：

- GitLab で Webhook の登録が有効になっていること。
- Webhook の登録に正しい URL とシークレットが設定されていること。

問題が解決しない場合は、サポートチーム（support@getdbt.com）までお問い合わせください。喜んでお手伝いいたします。
