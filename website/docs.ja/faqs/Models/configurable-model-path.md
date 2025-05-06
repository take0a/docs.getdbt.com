---
title: プロジェクトの `models` ディレクトリ以外のディレクトリにモデルを保存できますか?
description: "モデルディレクトリの命名方法"
sidebar_label: 'モデルディレクトリの命名方法'
id: configurable-model-path

---

デフォルトでは、dbt はモデルを定義するファイルがプロジェクトの `models` サブディレクトリにあることを想定しています。

これを変更するには、`dbt_project.yml` ファイルの [model-paths](reference/project-configs/model-paths.md) 設定を次のように更新します:

<File name='dbt_project.yml'>

```yml
model-paths: ["transformations"]
```

</File>
