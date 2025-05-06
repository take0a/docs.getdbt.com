---
title: プロジェクトの `tests` ディレクトリ以外のディレクトリにテストを保存できますか?
description: "テストを保存するディレクトリの場所"
sidebar_label: 'テストディレクトリの命名方法'
id: configurable-data-test-path

---
デフォルトでは、dbt は個別のテストファイルをプロジェクトの `tests` サブディレクトリに配置し、汎用テスト定義を `tests/generic` または `macros` に配置することを想定しています。

これを変更するには、`dbt_project.yml` ファイルの [test-paths](reference/project-configs/test-paths.md) 設定を次のように更新します。

<File name='dbt_project.yml'>

```yml
test-paths: ["my_cool_tests"]
```

</File>

それから、`my_cool_tests/generic/` に汎用テストを定義し、`my_cool_tests/` 内の他のすべての場所に特異テストを定義します。
