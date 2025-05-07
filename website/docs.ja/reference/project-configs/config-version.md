---
datatype: integer
description: "dbt の config-version 構成を理解するには、このガイドをお読みください。"
---

`config-version:` タグはオプションです。

<File name='dbt_project.yml'>

```yml
config-version: 2
```

</File>

## 定義

`dbt_project.yml` を v2 構造を使用して指定します。

## デフォルト

この設定がない場合、dbt は `dbt_project.yml` がバージョン 2 の構文を使用していると想定します。バージョン 1 は非推奨です。
