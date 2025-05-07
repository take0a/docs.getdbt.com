---
title: dispatch (config)
description: "dbt のディスパッチ構成を理解するには、このガイドをお読みください。"
datatype: list
required: False
---

<File name='dbt_project.yml'>

```yml
dispatch:
  - macro_namespace: packagename
    search_order: [packagename]
  - macro_namespace: packagename
    search_order: [packagename]
```

</File>

## 定義

必要に応じて、[dispatch](/reference/dbt-jinja-functions/dispatch) による特定の名前空間内のマクロの検索場所をオーバーライドします。指定されていない場合、`dispatch` はデフォルトでまずルートプロジェクトを検索し、次に `macro_namespace` で指定されたパッケージ内の実装を検索します。

## 例

`dbt_utils` パッケージを `spark_utils` 互換パッケージで「shim」したいとします。

<File name='dbt_project.yml'>

```yml
dispatch:
  - macro_namespace: dbt_utils
    search_order: ['spark_utils', 'dbt_utils']
```

</File>

ルートプロジェクト（`'my_root_project'`）で `dbt_utils` パッケージの特定のマクロを再実装しましたが、自分のバージョンを優先させたいと考えています。そうでない場合は、`dbt_utils` のバージョンにフォールバックします。

_注: これはデフォルトの動作です。必要に応じて、検索順序を次のように明示的に指定することもできます。_

<File name='dbt_project.yml'>

```yml
dispatch:
  - macro_namespace: dbt_utils
    search_order: ['my_root_project', 'dbt_utils']
```

</File>
