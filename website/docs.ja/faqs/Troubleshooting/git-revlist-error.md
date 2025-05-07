---
title: IDE で git rev-list master エラーが表示されますか?
description: "プライマリブランチが認識されません"
sidebar_label: 'IDEでgit rev-list masterエラーが発生する'
id: git-revlist-error
---

以下のエラー メッセージが表示されて IDE にアクセスできない場合は、以下の手順に従って問題を解決できるよう最善を尽くします。

```shell
git rev-list master..origin/main --count
fatal: ambiguous argument 'master..origin/main': unknown revision or path not in the working tree.
Use '--' to separate paths from revisions, like this:
'git <command> [<revision>...] -- [<file>...]'
```

通常、このエラーは「メイン」ブランチ名が変更されたか、dbt Cloud がプライマリブランチを特定できなかったことを示しています。ご安心ください。いくつかの回避策をお試しください。

**回避策 1**
環境設定を確認してください。環境設定にカスタムブランチが**入力されていない**場合：

1. プロジェクト設定ページでリポジトリの [接続](https://docs.getdbt.com/docs/dbt-cloud/cloud-configuring-dbt-cloud/cloud-import-a-project-by-git-url) を切断し、再接続してください。これにより、dbt Cloud は「メイン」ブランチの名前が「main」になったことを認識できるようになります。
2. 環境設定で、カスタムブランチを「master」に設定し、IDE を更新します。

**回避策 2**
環境設定を確認します。環境設定にカスタムブランチが設定されている場合は、以下の手順を実行してください。

1. プロジェクト設定ページでリポジトリの [接続](https://docs.getdbt.com/docs/dbt-cloud/cloud-configuring-dbt-cloud/cloud-import-a-project-by-git-url) を切断し、再接続します。これにより、dbt Cloud は「メイン」ブランチの名前が「main」になったことを認識できるようになります。
2. 環境設定で、カスタムブランチを削除し、IDE を更新します。

上記の回避策を試しても問題が解決しない場合は、サポートチーム（support@getdbt.com）までご連絡ください。喜んでお手伝いいたします。
