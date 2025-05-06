---
title: プロジェクトの `seeds` ディレクトリ以外のディレクトリにシードを保存できますか?
description: "シードを保存するディレクトリの場所"
sidebar_label: 'シードディレクトリの命名方法'
id: configurable-data-path

---

デフォルトでは、dbt はシードファイルがプロジェクトの `seeds` サブディレクトリにあることを想定しています。

これを変更するには、`dbt_project.yml` ファイルの [seed-paths](reference/project-configs/seed-paths.md) 設定を次のように更新します。

<File name='dbt_project.yml'>

```yml
seed-paths: ["custom_seeds"]
```

</File>
