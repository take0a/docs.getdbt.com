---
title: "増分戦略について"
sidebar_label: "Incremental strategy"
description: "マテリアライゼーションの増分戦略は、新しいデータと変更されたデータの処理方法を定義することでパフォーマンスを最適化します。"
id: "incremental-strategy"
intro_text: "マテリアライゼーションの増分戦略は、新しいデータと変更されたデータの処理方法を定義することでパフォーマンスを最適化します。"
---

増分マテリアライゼーションの概念を実装するには、様々な戦略があります。それぞれの戦略の価値は、以下の要素に依存します。

* データ量。
* `unique_key` の信頼性。
* データプラットフォームにおける特定の機能のサポート状況。

一部のアダプタでは、dbt が増分モデルの構築に使用するコードを制御するためのオプションの `incremental_strategy` 構成が提供されています。

:::info マイクロバッチ

[`microbatch` 増分戦略](/docs/build/incremental-microbatch) は、大規模な時系列データセットを対象としています。dbt は、設定された `event_time` 列に基づいて、増分モデルを複数のクエリ（または「バッチ」）で処理します。データの量と性質によっては、新しいデータを追加するために単一のクエリを使用するよりも、この方法の方が効率的で回復力に優れている場合があります。

:::

### アダプタ別にサポートされている増分戦略

この表は、dbt Cloud の [最新リリーストラック](/docs/dbt-versions/cloud-release-tracks) で利用可能なアダプタにおける各増分戦略のサポート状況を示しています。「最新」ではなく、機能が「互換」トラックにリリースされていない場合、一部の戦略は利用できない場合があります。

dbt Core でのみ利用可能なアダプタにご興味がある場合は、[アダプタの個別の構成ページ](/reference/resource-configs/resource-configs) で詳細をご確認ください。

サポートされている増分戦略の詳細については、次の表でアダプタ名をクリックしてください:

| Data platform adapter | `append` | `merge` | `delete+insert` | `insert_overwrite` | `microbatch`        |
|-----------------------|:--------:|:-------:|:---------------:|:------------------:|:-------------------:|
| [dbt-postgres](/reference/resource-configs/postgres-configs#incremental-materialization-strategies) |     ✅    |    ✅   |  ✅ |   |   ✅   |
| [dbt-redshift](/reference/resource-configs/redshift-configs#incremental-materialization-strategies) |     ✅    |    ✅   |  ✅ |   |   ✅   |
| [dbt-bigquery](/reference/resource-configs/bigquery-configs#merge-behavior-incremental-models)      |           |    ✅   |    | ✅ |  ✅    |
| [dbt-spark](/reference/resource-configs/spark-configs#incremental-models)                           |     ✅    |    ✅   |    |    ✅   | ✅ |
| [dbt-databricks](/reference/resource-configs/databricks-configs#incremental-models)                 |     ✅    |    ✅   |    |          ✅         |          ✅         |
| [dbt-snowflake](/reference/resource-configs/snowflake-configs#merge-behavior-incremental-models)    |     ✅    |    ✅   | ✅  | ✅ | ✅  |
| [dbt-trino](/reference/resource-configs/trino-configs#incremental)                                  |     ✅    |    ✅   | ✅  |    |    |
| [dbt-fabric](/reference/resource-configs/fabric-configs#incremental)                                |     ✅    |         | ✅  |    |    |
| [dbt-athena](/reference/resource-configs/athena-configs#incremental-models)                         |     ✅    |    ✅   |     | ✅ |    |
| [dbt-teradata](/reference/resource-configs/teradata-configs#valid_history-incremental-materialization-strategy)  | ✅    |  ✅   |   ✅   |    |         ✅    |

### 増分戦略の設定

`incremental_strategy` 設定は、`dbt_project.yml` ファイル内の特定のモデルまたはすべてのモデルに対して定義できます:

<File name='dbt_project.yml'>

```yaml
models:
  +incremental_strategy: "insert_overwrite"
```

</File>

または:

<File name='models/my_model.sql'>

```sql
{{
  config(
    materialized='incremental',
    unique_key='date_day',
    incremental_strategy='delete+insert',
    ...
  )
}}

select ...
```

</File>

### 戦略固有の設定

`merge` 戦略を使用し、`unique_key` を指定した場合、デフォルトでは、dbt は一致した行全体を新しい値で上書きします。

`merge` 戦略をサポートするアダプタ（Snowflake、BigQuery、Apache Spark、Databricks など）では、オプションで列名のリストを `merge_update_columns` 設定に渡すことができます。その場合、dbt は設定で指定された列のみを更新し、他の列は以前の値を保持します。

<File name='models/my_model.sql'>

```sql
{{
  config(
    materialized = 'incremental',
    unique_key = 'id',
    merge_update_columns = ['email', 'ip_address'],
    ...
  )
}}

select ...
```

</File>

あるいは、列名のリストを `merge_exclude_columns` 設定に渡すことで、更新から除外する列のリストを指定することもできます。

<File name='models/my_model.sql'>

```sql
{{
  config(
    materialized = 'incremental',
    unique_key = 'id',
    merge_exclude_columns = ['created_at'],
    ...
  )
}}

select ...
```

</File>

### incremental_predicates について

`incremental_predicates` は、データ量が多くパフォーマンスへの追加投資が正当化されるような増分モデルの高度な使用方法です。この設定は、有効な SQL 式のリストを受け入れます。dbt は SQL 文の構文をチェックしません。

これは、Snowflake でよく見られる `yml` ファイル内のモデル設定の例です:

```yml

models:
  - name: my_incremental_model
    config:
      materialized: incremental
      unique_key: id
      # this will affect how the data is stored on disk, and indexed to limit scans
      cluster_by: ['session_start']  
      incremental_strategy: merge
      # this limits the scan of the existing table to the last 7 days of data
      incremental_predicates: ["DBT_INTERNAL_DEST.session_start > dateadd(day, -7, current_date)"]
      # `incremental_predicates` accepts a list of SQL statements. 
      # `DBT_INTERNAL_DEST` and `DBT_INTERNAL_SOURCE` are the standard aliases for the target table and temporary table, respectively, during an incremental run using the merge strategy. 
```

あるいは、モデル ファイル内で構成された同じ構成は次のとおりです:

```sql
-- in models/my_incremental_model.sql

{{
  config(
    materialized = 'incremental',
    unique_key = 'id',
    cluster_by = ['session_start'],  
    incremental_strategy = 'merge',
    incremental_predicates = [
      "DBT_INTERNAL_DEST.session_start > dateadd(day, -7, current_date)"
    ]
  )
}}

...

```

これにより、次のような `merge` ステートメントが (`dbt.log` ファイル内に) テンプレート化されます:

```sql
merge into <existing_table> DBT_INTERNAL_DEST
    from <temp_table_with_new_records> DBT_INTERNAL_SOURCE
    on
        -- unique key
        DBT_INTERNAL_DEST.id = DBT_INTERNAL_SOURCE.id
        and
        -- custom predicate: limits data scan in the "old" data / existing table
        DBT_INTERNAL_DEST.session_start > dateadd(day, -7, current_date)
    when matched then update ...
    when not matched then insert ...
```

増分モデル SQL 本体内の上流テーブルのデータ スキャンを制限します。これにより、処理/変換される「新しい」データの量が制限されます。

```sql
with large_source_table as (

    select * from {{ ref('large_source_table') }}
    {% if is_incremental() %}
        where session_start >= dateadd(day, -3, current_date)
    {% endif %}

),

...
```

:::info
構文は、`incremental_strategy` の設定方法によって異なります:
- `merge` 戦略を使用する場合、列に `DBT_INTERNAL_DEST`（「古い」データ）または `DBT_INTERNAL_SOURCE`（「新しい」データ）のいずれかを明示的に別名設定する必要がある場合があります。
- `insert_overwrite` 増分戦略と概念的に重複する部分がかなりあります。
:::

### 組み込み戦略

[カスタム戦略](#custom-strategies) について詳しく説明する前に、dbt に組み込まれている増分戦略とそれに対応するマクロについて理解しておくことが重要です。

| `incremental_strategy` | Corresponding macro                    |
|------------------------|----------------------------------------|
| `append`               | `get_incremental_append_sql`           |
| `delete+insert`        | `get_incremental_delete_insert_sql`    |
| `merge`                | `get_incremental_merge_sql`            |
| `insert_overwrite`     | `get_incremental_insert_overwrite_sql` |
| `microbatch`           | `get_incremental_microbatch_sql`       |


たとえば、`append` の組み込み戦略は、次のファイルで定義して使用できます:

<File name='macros/append.sql'>

```sql
{% macro get_incremental_append_sql(arg_dict) %}

  {% do return(some_custom_macro_with_sql(arg_dict["target_relation"], arg_dict["temp_relation"], arg_dict["unique_key"], arg_dict["dest_columns"], arg_dict["incremental_predicates"])) %}

{% endmacro %}


{% macro some_custom_macro_with_sql(target_relation, temp_relation, unique_key, dest_columns, incremental_predicates) %}

    {%- set dest_cols_csv = get_quoted_csv(dest_columns | map(attribute="name")) -%}

    insert into {{ target_relation }} ({{ dest_cols_csv }})
    (
        select {{ dest_cols_csv }}
        from {{ temp_relation }}
    )

{% endmacro %}
```
</File>

モデル models/my_model.sql を定義します:

```sql
{{ config(
    materialized="incremental",
    incremental_strategy="append",
) }}

select * from {{ ref("some_model") }}
```

### カスタム戦略

:::note limited support

カスタム戦略は現在、BigQuery および Spark アダプタではサポートされていません。

:::

dbt v1.2 以降では、[全く新しいマテリアライゼーションを作成する](/guides/create-new-materializations)よりも簡単な方法が利用できます。ユーザーは以下の方法で独自の「カスタム」増分戦略を定義し、使用できます。

1. `get_incremental_STRATEGY_sql` というマクロを定義します。`STRATEGY` はプレースホルダなので、カスタム増分戦略の名前に置き換えてください。
2. 増分モデル内で `incremental_strategy: STRATEGY` を設定します。

dbt はユーザー定義の戦略を検証せず、その名前のマクロを検索し、見つからない場合はエラーを発生させます。

例えば、`insert_only` というユーザー定義戦略は、以下のファイルで定義して使用できます。

<File name='macros/my_custom_strategies.sql'>

```sql
{% macro get_incremental_insert_only_sql(arg_dict) %}

  {% do return(some_custom_macro_with_sql(arg_dict["target_relation"], arg_dict["temp_relation"], arg_dict["unique_key"], arg_dict["dest_columns"], arg_dict["incremental_predicates"])) %}

{% endmacro %}


{% macro some_custom_macro_with_sql(target_relation, temp_relation, unique_key, dest_columns, incremental_predicates) %}

    {%- set dest_cols_csv = get_quoted_csv(dest_columns | map(attribute="name")) -%}

    insert into {{ target_relation }} ({{ dest_cols_csv }})
    (
        select {{ dest_cols_csv }}
        from {{ temp_relation }}
    )

{% endmacro %}
```

</File>

<File name='models/my_model.sql'>

```sql
{{ config(
    materialized="incremental",
    incremental_strategy="insert_only",
    ...
) }}

...
```

</File>

カスタム マイクロバッチ マクロを使用する場合は、`dbt_project.yml` で [`require_batched_execution_for_custom_microbatch_strategy` 動作フラグ](/reference/global-configs/behavior-changes#custom-microbatch-strategy) を設定して、カスタム戦略のバッチ実行を有効にします。

### パッケージからのカスタム戦略

`example` パッケージの `merge_null_safe` カスタム増分戦略を使用するには、以下の手順に従います。
- [パッケージをインストール](/docs/build/packages#how-do-i-add-a-package-to-my-project)
- 次のマクロをプロジェクトに追加します。

<File name='macros/my_custom_strategies.sql'>

```sql
{% macro get_incremental_merge_null_safe_sql(arg_dict) %}
    {% do return(example.get_incremental_merge_null_safe_sql(arg_dict)) %}
{% endmacro %}
```

</File>

<Snippet path="discourse-help-feed-header" />
<DiscourseHelpFeed tags="incremental"/>
