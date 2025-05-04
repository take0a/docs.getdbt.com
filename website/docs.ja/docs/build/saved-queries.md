---
title: 保存済みクエリ
id: saved-queries
description: "保存済みクエリは、MetricFlowでよく使用するクエリを保存する方法です。これにより時間を節約し、同じクエリを何度も記述する必要がなくなります。"
sidebar_label: "Saved queries"
tags: [Metrics, Semantic Layer]
---

保存済みクエリは、MetricFlowでよく使用するクエリを保存する方法です。論理的に関連する指標、ディメンション、フィルターをグループ化して、保存済みクエリを作成できます。保存済みクエリはノードとして、dbt <Term id="dag" /> に表示されます。

保存済みクエリは基本的な構成要素として機能し、保存済みクエリ設定で[エクスポートを構成](#configure-exports)できます。エクスポートではこの機能がさらに強化され、[dbt Cloud のジョブスケジューラ](/docs/deploy/job-scheduler)を使用してデータプラットフォーム内で直接[保存済みクエリのスケジュール設定と書き込み](/docs/use-dbt-semantic-layer/exports)が可能になります。

## パラメータ

保存クエリを作成するには、次の表のパラメータを参照してください。

:::tip
パラメータが別のパラメータ内にネストされているかどうかを示すために、二重コロン (::) を使用することに注意してください。たとえば、`query_params::metrics` は、`metrics` パラメータが `query_params` の下にネストされていることを意味します。
:::

<!-- For versions 1.9 and higher -->
<VersionBlock firstVersion="1.9">

| Parameter | Type    | Required | Description    |
|-------|---------|----------|----------------|
| `name`       | String    | Required     | 保存されたクエリ オブジェクトの名前。 |
| `description`     | String      | Required     | 保存されたクエリの説明。 |
| `label`     | String      | Required     | 保存したクエリの表示名。この値は下流のツールに表示されます。 |
| `config`     | String      |  Optional     | [`config`](/reference/resource-properties/config) プロパティを使用して、保存したクエリの設定を指定します。`cache`、[`enabled`](/reference/resource-configs/enabled)、`export_as`、[`group`](/reference/resource-configs/group)、[`meta`](/reference/resource-configs/meta)、[`tags`](/reference/resource-configs/tags)、[`schema`](/reference/resource-configs/schema) の設定がサポートされています。 |
| `config::cache::enabled`     | Object      | Optional     | 保存されたクエリを[キャッシュ](/docs/use-dbt-semantic-layer/sl-cache)に読み込むかどうかを指定するために使用するサブキーを持つオブジェクト。サブキーは`true`または`false`です。デフォルトは`false`です。 |
| `query_params`       | Structure   | Required     | クエリパラメータが含まれます。 |
| `query_params::metrics`   | List or String   | Optional    | コマンドライン インターフェイスで指定されたクエリで使用されるメトリックのリスト。 |
| `query_params::group_by`    | List or String          | Optional    | クエリで使用されるエンティティとディメンションのリスト。これには `Dimension` または `TimeDimension` が含まれます。 |
| `query_params::where`        | List or String | Optional  | `Dimension` または `TimeDimension` オブジェクトを含む可能性のある文字列のリスト。 |
| `exports`     | List or Structure | Optional    | エクスポート構造内で指定されるエクスポートのリスト。 |
| `exports::name`       | String               | Required     | エクスポート オブジェクトの名前。 |
| `exports::config`     | List or Structure     | Required     | エクスポートを指定するパラメータの [`config`](/reference/resource-properties/config) プロパティ。 |
| `exports::config::export_as` | String    | Required     | 実行するエクスポートの種類。オプションには、現在テーブルまたはビュー、近い将来キャッシュが含まれます。 |
| `exports::config::schema`   | String   | Optional    | テーブルまたはビューを作成するための[スキーマ](/reference/resource-configs/schema)。このオプションはキャッシュには使用できません。 |
| `exports::config::alias`  | String     | Optional    | テーブルまたはビューへの書き込みに使用されるテーブル[エイリアス](/reference/resource-configs/alias)。このオプションはキャッシュには使用できません。 | 

</VersionBlock>

<!-- For versions 1.8 and higher -->
<VersionBlock firstVersion="1.8" lastVersion="1.8">

| Parameter | Type    | Required | Description    |
|-------|---------|----------|----------------|
| `name`       | String    | Required     | Name of the saved query object.          |
| `description`     | String      | Required     | A description of the saved query.     |
| `label`     | String      | Required     | The display name for your saved query. This value will be shown in downstream tools.    |
| `config`     | String      |  Optional     |  Use the [`config`](/reference/resource-properties/config) property to specify configurations for your saved query. Supports `cache`, [`enabled`](/reference/resource-configs/enabled), `export_as`, [`group`](/reference/resource-configs/group), [`meta`](/reference/resource-configs/meta), and [`schema`](/reference/resource-configs/schema)  configurations.   |
| `config::cache::enabled`     | Object      | Optional     |  An object with a sub-key used to specify if a saved query should populate the [cache](/docs/use-dbt-semantic-layer/sl-cache). Accepts sub-key `true` or `false`. Defaults to `false` |
| `query_params`       | Structure   | Required     | Contains the query parameters. |
| `query_params::metrics`   | List or String   | Optional    | A list of the metrics to be used in the query as specified in the command line interface. |
| `query_params::group_by`    | List or String          | Optional    | A list of the Entities and Dimensions to be used in the query, which include the `Dimension` or `TimeDimension`. |
| `query_params::where`        | List or String | Optional  | A list of strings that may include the `Dimension` or `TimeDimension` objects. |
| `exports`     | List or Structure | Optional    | A list of exports to be specified within the exports structure.     |
| `exports::name`       | String               | Required     | Name of the export object.      |
| `exports::config`     | List or Structure     | Required     | A [`config`](/reference/resource-properties/config) property for any parameters specifying the export.  |
| `exports::config::export_as` | String    | Required     | The type of export to run. Options include table or view currently and cache in the near future.   |
| `exports::config::schema`   | String   | Optional    | The schema for creating the table or view. This option cannot be used for caching.   |
| `exports::config::alias`  | String     | Optional    | The table alias used to write to the table or view.  This option cannot be used for caching.  | 

</VersionBlock> 

<!-- For versions 1.7 and lower-->
<VersionBlock firstVersion="1.7" lastVersion="1.7">

| Parameter | Type    | Required | Description    |
|-------|---------|----------|----------------|
| `name`       | String    | Required     | Name of the saved query object.          |
| `description`     | String      | Required     | A description of the saved query.     |
| `label`     | String      | Required     | The display name for your saved query. This value will be shown in downstream tools.    |
| `query_params`       | Structure   | Required     | Contains the query parameters. |
| `query_params::metrics`   | List or String   | Optional    | Metrics nested with the `query_params`: a list of the metrics to be used in the query as specified in the command line interface. |
| `query_params::group_by`    | List or String          | Optional    | Grouping nested with the `query_params`: a list of the Entities and Dimensions to be used in the query, which include the `Dimension` or `TimeDimension`. |
| `query_params::where`        | List or String | Optional  | Conditions nested with the `query_params`: a list of strings that may include the `Dimension` or `TimeDimension` objects. |
| `exports`     | List or Structure | Optional    | A list of exports to be specified within the exports structure.     |
| `exports::name`       | String               | Required     | Name of export object, nested within `exports`.   |
| `exports::config`     | List or Structure     | Required     | A [`config`](/reference/resource-properties/config) property for any parameters specifying the export, nested within `exports`.  |
| `exports::config::export_as` | String    | Required     |  Specifies the type of export: table, view, or upcoming cache options. Nested within `exports` and `config`.   |
| `exports::config::schema`   | String   | Optional    | Schema for creating the table or view, not applicable for caching. Nested within `exports` and `config`.   |
| `exports::config::alias`  | String     | Optional    | Table alias used to write to the table or view.  This option can't be used for caching. Nested within `exports` and `config`.  |

</VersionBlock>

保存済みクエリで複数の指標を使用する場合、`group_by` 句または `where` 句では、これらの指標に共通するディメンションのみを参照できます。ディメンションオブジェクトには、`Dimension('user__ds')` のように、エンティティ名プレフィックスを使用してください。

## 保存済みクエリの設定

保存済みクエリを使用して、指標やディメンションを含む一般的なセマンティックレイヤークエリをYAMLで定義および管理できます。保存済みクエリを使用すると、dbtプロジェクト内でよく使用されるMetricFlowクエリを整理して再利用できます。たとえば、関連する指標をグループ化して整理しやすくしたり、よく使用するディメンションやフィルターを含めたりできます。

保存済みクエリの設定では、dbt Cloudジョブスケジューラで[キャッシュ](/docs/use-dbt-semantic-layer/sl-cache)を活用することで、よく使用するクエリをキャッシュし、パフォーマンスを向上させ、コンピューティングコストを削減することもできます。

<!-- For versions 1.9 and higher -->

次の例では、保存されたクエリを `semantic_model.yml` ファイルに設定できます:

<File name='semantic_model.yml'>
<VersionBlock firstVersion="1.9">

```yaml
saved_queries:
  - name: test_saved_query
    description: "{{ doc('saved_query_description') }}"
    label: Test saved query
    config:
      cache:
        [enabled](/reference/resource-configs/enabled): true | false
        [tags](/reference/resource-configs/tags): 'my_tag'
    query_params:
      metrics:
        - simple_metric
      group_by:
        - "Dimension('user__ds')"
      where:
        - "{{ Dimension('user__ds', 'DAY') }} <= now()"
        - "{{ Dimension('user__ds', 'DAY') }} >= '2023-01-01'"
    exports:
      - name: my_export
        config:
          export_as: table 
          alias: my_export_alias
          schema: my_export_schema_name
```
</VersionBlock>

<!-- For versions 1.8 and higher -->
<VersionBlock firstVersion="1.8" lastVersion="1.8">

```yaml
saved_queries:
  - name: test_saved_query
    description: "{{ doc('saved_query_description') }}"
    label: Test saved query
    config:
      cache:
        enabled: true  # Or false if you want it disabled by default
    query_params:
      metrics:
        - simple_metric
      group_by:
        - "Dimension('user__ds')"
      where:
        - "{{ Dimension('user__ds', 'DAY') }} <= now()"
        - "{{ Dimension('user__ds', 'DAY') }} >= '2023-01-01'"
    exports:
      - name: my_export
        config:
          export_as: table
          alias: my_export_alias
          schema: my_export_schema_name
```

</VersionBlock>
</File>

<VersionBlock firstVersion="1.8">

なお、保存済みクエリとエクスポートの [config](/reference/resource-properties/config) の両方に `export_as` を設定できますが、エクスポートの設定値が優先されます。エクスポートの設定にキーが設定されていない場合は、保存済みクエリの設定値が継承されます。

#### Where句

フィルター内のエンティティ、ディメンション、時間ディメンション、または指標を参照するには、次の構文を使用します。指標フィルターで指標をディメンションとして使用する方法の詳細については、[ディメンションとしての指標](/docs/build/ref-metrics-in-filters)を参照してください。

```yaml
filter: | 
  {{ Entity('entity_name') }}

filter: |  
  {{ Dimension('primary_entity__dimension_name') }}

filter: |  
  {{ TimeDimension('time_dimension', 'granularity') }}

filter: |  
  {{ Metric('metric_name', group_by=['entity_name']) }}
```
</VersionBlock>

<!-- For versions 1.7 and lower-->
<VersionBlock lastVersion="1.7">

In the following example, you can set the saved query in the `semantic_model.yml` file:

<File name='semantic_model.yml'>

```yaml
saved_queries:
  - name: test_saved_query
    description: "{{ doc('saved_query_description') }}"
    label: Test saved query
    query_params:
      metrics:
        - simple_metric
      group_by:
        - "Dimension('user__ds')"
      where:
        - "{{ Dimension('user__ds', 'DAY') }} <= now()"
        - "{{ Dimension('user__ds', 'DAY') }} >= '2023-01-01'"
    exports:
      - name: my_export
        config:
          export_as: table
          alias: my_export_alias
          schema: my_export_schema_name
```
</File>
</VersionBlock>

#### プロジェクトレベルの保存済みクエリ

プロジェクトレベルで保存済みクエリを有効にするには、[`dbt_project.yml` ファイル](/reference/dbt_project.yml) で `saved-queries` 設定を設定します。これにより、各ファイルで保存済みクエリを設定する手間が省けます。

<File name='dbt_project.yml'>

```yaml
saved-queries:
  my_saved_query:
    +cache:
      enabled: true
```
</File>

`dbt_project.yml` と設定の命名規則の詳細については、[dbt_project.yml リファレンスページ](/reference/dbt_project.yml#naming-convention) を参照してください。

`saved_queries` をビルドするには、[`--resource-type` フラグ](/reference/global-configs/resource-type) を使用して、コマンド `dbt build --resource-type saved_query` を実行します。

## エクスポートの設定

エクスポートは、保存済みクエリに追加される設定です。保存済みクエリの記述方法、スキーマ、テーブル名を定義します。

保存済みクエリの設定と基盤ブロックの設定が完了したら、`saved_queries` YAML 設定ファイル（指標定義と同じファイル）でエクスポートを設定できるようになります。これにより、[dbt Cloud のジョブスケジューラ](/docs/deploy/job-scheduler) を使用して、データプラットフォーム内で [エクスポートを自動実行](#run-exports) することも可能になります。

以下はエクスポートで保存されたクエリの例です:

<File name='semantic_model.yml'>
<VersionBlock firstVersion="1.9">

```yaml
saved_queries:
  - name: order_metrics
    description: Relevant order metrics
    config:
      tags:
        - order_metrics
    query_params:
      metrics:
        - orders
        - large_order
        - food_orders
        - order_total
      group_by:
        - Entity('order_id')
        - TimeDimension('metric_time', 'day')
        - Dimension('customer__customer_name')
        - ... # Additional group_by
      where:
        - "{{TimeDimension('metric_time')}} > current_timestamp - interval '1 week'"
         - ... # Additional where clauses
    exports:
      - name: order_metrics
        config:
          export_as: table # Options available: table, view
          [alias](/reference/resource-configs/alias): my_export_alias # Optional - defaults to Export name
          [schema](/reference/resource-configs/schema): my_export_schema_name # Optional - defaults to deployment schema           
```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
saved_queries:
  - name: order_metrics
    description: Relevant order metrics
    query_params:
      metrics:
        - orders
        - large_order
        - food_orders
        - order_total
      group_by:
        - Entity('order_id')
        - TimeDimension('metric_time', 'day')
        - Dimension('customer__customer_name')
        - ... # Additional group_by
      where:
        - "{{TimeDimension('metric_time')}} > current_timestamp - interval '1 week'"
         - ... # Additional where clauses
    exports:
      - name: order_metrics
        config:
          export_as: table # Options available: table, view
          schema: my_export_schema_name # Optional - defaults to deployment schema
          alias: my_export_alias # Optional - defaults to Export name
```
</VersionBlock>

</File>

## エクスポートの実行

エクスポートの設定が完了したら、[dbt Cloud のジョブスケジューラ](/docs/deploy/job-scheduler)を使用してエクスポートを実行し、保存済みのクエリをデータプラットフォーム内に自動的に書き込むことで、さらに高度な機能を実現できます。この機能は、[dbt Cloud のセマンティックレイヤー](/docs/use-dbt-semantic-layer/dbt-sl)でのみ利用できます。

エクスポートの実行方法の詳細については、[エクスポート](/docs/use-dbt-semantic-layer/exports)のドキュメントをご覧ください。

## FAQs

<DetailsToggle alt_header="1 つの保存されたクエリで複数のエクスポートを実行できますか?">

はい、可能です。ただし、エクスポートの名前、スキーマ、およびマテリアライズ戦略が異なります。
</DetailsToggle>

<DetailsToggle alt_header="リソースタイプ別に saved_queries を選択するにはどうすればよいですか?">

保存されているすべてのクエリを dbt ビルド実行に含めるには、[`--resource-type` フラグ](/reference/global-configs/resource-type) を使用して、コマンド `dbt build --resource-type saved_query` を実行します。

</DetailsToggle>

## 関連ドキュメント
- [CI ジョブでセマンティックノードを検証する](/docs/deploy/ci-jobs#semantic-validations-in-ci)
- [キャッシュ](/docs/use-dbt-semantic-layer/sl-cache) を設定する
