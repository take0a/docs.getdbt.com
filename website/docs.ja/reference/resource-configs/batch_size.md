---
title: "batch_size"
id: "batch-size"
sidebar_label: "batch_size"
resource_types: [models]
description: "dbt は、マイクロバッチ増分モデルを実行するときに、バッチの大きさを決定するために `batch_size` を使用します。"
datatype: hour | day | month | year
---

<VersionCallout version="1.9" />

## 定義

`batch_size` 設定は、[マイクロバッチ増分モデル](/docs/build/incremental-microbatch) を実行する際のバッチサイズを決定します。指定できる値は、`hour`、`day`、`month`、または `year` です。[モデル](/docs/build/models) の `batch_size` は、`dbt_project.yml` ファイル、プロパティ YAML ファイル、または設定ブロックで設定できます。

## 例

以下の例では、`user_sessions` モデルの `batch_size` として `day` を設定します。

`dbt_project.yml` ファイル内の `batch_size` 設定の例:

<File name='dbt_project.yml'>

```yml
models:
  my_project:
    user_sessions:
      +batch_size: day
```
</File>

プロパティ YAML ファイルの例:

<File name='models/properties.yml'>

```yml
models:
  - name: user_sessions
    config:
      batch_size: day
```

</File>

SQL モデル構成ブロックの例:

<File name="models/user_sessions.sql">

```sql
{{ config(
    materialized='incremental',
    batch_size='day'
) }}
```

</File> 

