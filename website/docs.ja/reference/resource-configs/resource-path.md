---
title: Resource path
description: "リソース パスを使用して dbt でリソース タイプを構成する方法を学習します。"
id: resource-path
sidebar_label: "About resource paths"
---

このドキュメントでは、`<resource-path>` という命名法を使用して、`dbt_project.yml` ファイルからモデル、シード、スナップショット、テスト、ソースなどのリソースタイプを設定する方法を説明します。

これは、そのリソースタイプのディレクトリへのパス、または名前でそのリソースタイプの単一インスタンスを提供する、ネストされた辞書キーを表します。

```yml
resource_type:
  project_name:
    directory_name:
      subdirectory_name:
        instance_of_resource_type (by name):
          ...
```

## 例

以下の例は主にモデルとソースを対象としていますが、シード、スナップショット、テスト、ソース、その他のリソースタイプにも同じ概念が適用されます。

### すべてのモデルに設定を適用する

すべてのモデルに設定を適用するには、`<resource-path>` を使用しないでください。

<File name='dbt_project.yml'>

```yml
models:
  +enabled: false # this will disable all models (not a thing you probably want to do)
```

</File>

### プロジェクト内のすべてのモデルに設定を適用する

プロジェクト内のすべてのモデルに設定を適用するには、[プロジェクト名](/reference/project-configs/name) を `<resource-path>` として使用します。

<File name='dbt_project.yml'>

```yml
name: jaffle_shop

models:
  jaffle_shop:
    +enabled: false # this will apply to all models in your project, but not any installed packages
```

</File>

### サブディレクトリ内のすべてのモデルに設定を適用する

プロジェクトのサブディレクトリ（例：`staging`）内のすべてのモデルに設定を適用するには、プロジェクト名の下にディレクトリをネストします:

<File name='dbt_project.yml'>

```yml
name: jaffle_shop

models:
  jaffle_shop:
    staging:
      +enabled: false # this will apply to all models in the `staging/` directory of your project
```

</File>

次のプロジェクトでは、これは `staging/` ディレクトリ内のモデルに適用されますが、`marts/` ディレクトリには適用されません:

```
.
├── dbt_project.yml
└── models
    ├── marts
    └── staging

```

### 特定のモデルに設定を適用する

特定のモデルに設定を適用するには、プロジェクト名の下にフルパスをネストします。`/staging/stripe/payments.sql` にあるモデルの場合、以下のようになります。

<File name='dbt_project.yml'>

```yml
name: jaffle_shop

models:
  jaffle_shop:
    staging:
      stripe:
        payments:
          +enabled: false # this will apply to only one model
```

</File>

次のプロジェクトでは、これは `payments` モデルにのみ適用されます:

```
.
├── dbt_project.yml
└── models
    ├── marts
    │   └── core
    │       ├── dim_customers.sql
    │       └── fct_orders.sql
    └── staging
        ├── jaffle_shop
        │   ├── customers.sql
        │   └── orders.sql
        └── stripe
            └── payments.sql

```
### サブフォルダにネストされたソースに構成を適用する

サブフォルダ内の YAML ファイルにネストされたソーステーブルを無効にするには、その YAML ファイルへのパスにサブフォルダを指定し、`dbt_project.yml` ファイルにソース名とテーブル名を指定する必要があります。<br /><br />
次の例は、サブフォルダ内の YAML ファイルにネストされたソーステーブルを無効にする方法を示しています。 

  <File name='dbt_project.yml'>

  ```yaml
  sources:
    your_project_name:
      subdirectory_name:
        source_name:
          source_table_name:
            +enabled: false
  ```
  </File>
