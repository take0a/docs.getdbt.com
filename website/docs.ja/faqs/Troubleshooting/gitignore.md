---
title: .gitignore ファイルを修正するにはどうすればいいですか?
description: "gitignoreファイルを修正するには、以下の手順を行ってください。"
sidebar_label: '.gitignore ファイルを修正する方法'
id: gitignore
---

gitignore ファイルは、Git が意図的に無視するファイルを指定します。プロジェクト内でこれらのファイルは、斜体で表示されるので識別できます。

変更を元に戻したり、ブランチをチェックアウトしたり、コミットをクリックしたりできない場合は、通常、プロジェクトに [.gitignore](https://github.com/dbt-labs/dbt-starter-project/blob/main/.gitignore) ファイルがないか、gitignore ファイルのフォルダ内に必要なコンテンツが含まれていないことが原因です。

この問題を解決するには、次の手順を実行します。

1. dbt Cloud IDE で、dbt プロジェクトの `.gitignore` ファイルに次の [.gitignore コンテンツ](https://github.com/dbt-labs/dbt-starter-project/blob/main/.gitignore) を追加します。

```bash
target/
dbt_packages/
logs/
# legacy -- renamed to dbt_packages in dbt v1
dbt_modules/
```
2. 変更を保存しますが、_コミットしないでください_。
3. IDE の右下にある **IDE Status button** の横にある 3 つのドットをクリックして、IDE を再起動します。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/restart-ide.jpg" width="50%" title="Restart the IDE by clicking the three dots on the lower right or click on the Status bar" />

4. **Restart IDE** を選択します。
5. dbt プロジェクトに戻り、以下のファイルまたはフォルダがある場合は削除します。
* `target`、`dbt_modules`、`dbt_packages`、`logs`
6. 変更を **Save** し、**Commit and sync** します。
7. IDE を再起動します。
8. 新しい変更を統合するには、**Version Control** メニューでプルリクエスト (PR) を作成します。
9. Git プロバイダーページで PR をマージします。
10. メインブランチに切り替え、**Pull from remote** をクリックして、メインブランチに加えたすべての変更をプルします。.gitignore ファイル内のファイル/フォルダが斜体になっていることを確認することで、変更を確認できます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/gitignore-italics.jpg" width="50%" title="A dbt project on the main branch that has properly configured gitignore folders (highlighted in italics)."/>

詳細については、こちらの[詳細なビデオ](https://www.loom.com/share/9b3b8e2b617f41a8bad76ec7e42dd014)を参照して追加のガイダンスを入手してください。
