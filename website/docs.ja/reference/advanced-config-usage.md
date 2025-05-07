---
title: 高度な構成の使用
sidebar_label: Advanced usage
---
## 代替の設定ブロック構文

一部の設定には、Jinja の引数として解析できない文字（例：ダッシュ）が含まれている場合があります。例えば、次の設定はエラーを返します。

```sql
{{ config(
    post-hook="grant select on {{ this }} to role reporter",
    materialized='table'
) }}

select ...
```

dbt はコア設定にエイリアスを提供します（例：設定ブロックでは `pre-hook` ではなく `pre_hook` を使用する必要があります）。ただし、dbt プロジェクトにはエイリアスのないカスタム設定が含まれている場合があります。

モデル内でこれらの設定を指定する場合は、代替の設定ブロック構文を使用します。


<File name='models/events/base/base_events.sql'>

```sql
{{
  config({
    "post-hook": "grant select on {{ this }} to role reporter",
    "materialized": "table"
  })
}}


select ...
```

</File>

<!---
## Hierarchies / overriding configs / precedence
For Drew to do
--->
