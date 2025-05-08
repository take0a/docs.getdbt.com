---
title: "concurrent_batches"
resource_types: [models]
datatype: model_name
description: "Learn about concurrent_batches in dbt."
---

<VersionCallout version="1.9" />

<Tabs>
<TabItem value="Project file">


<File name='dbt_project.yml'>

```yaml
models:
  +concurrent_batches: true
```

</File>

</TabItem>


<TabItem value="Config block">

<File name='models/my_model.sql'>

```sql
{{
  config(
    materialized='incremental',
    concurrent_batches=true,
    incremental_strategy='microbatch'
        ...
  )
}}
select ...
```

</File>

</TabItem>
</Tabs>

## 定義

`concurrent_batches` は、バッチを並列実行するか、それとも逐次（一度に 1 つずつ）実行するかを決定できるオーバーライドです。

詳細については、[バッチ実行の仕組み](/docs/build/parallel-batch-execution#how-parallel-batch-execution-works) を参照してください。

## 例

デフォルトでは、dbt はマイクロバッチモデルでバッチを並列実行できるかどうかを自動検出します。ただし、以下の条件を満たす場合は、`dbt_project.yml` ファイルまたはモデル `.sql` ファイルで `concurrent_batches` 構成を `false` に設定して並列実行または順次実行を指定することで、dbt の検出をオーバーライドできます。
* [マイクロバッチ増分戦略](/docs/build/incremental-microbatch) を設定している。
* 累積メトリクス、またはバッチ順序に依存するロジックを使用している。

バッチが順次処理されるようにするには、`concurrent_batches` 構成を `false` に設定します。例:

<File name='dbt_project.yml'>

```yaml
models:
  my_project:
    cumulative_metrics_model:
      +concurrent_batches: false
```
</File>


<File name='models/my_model.sql'>

```sql
{{
  config(
    materialized='incremental',
    incremental_strategy='microbatch'
    concurrent_batches=false
  )
}}
select ...

```
</File>


