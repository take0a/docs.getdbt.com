---
title: "マテリアライゼーション"
description: "dbt でマテリアライゼーションを構成して、SQL の実行方法と結果データの保存方法を制御します。"
id: "materializations"
pagination_next: "docs/build/incremental-models"
---

## 概要
<Term id="materialization">マテリアライゼーション</Term>は、dbt モデルをウェアハウスに永続化するための戦略です。dbt には 5 種類のマテリアライゼーションが組み込まれています。それらは以下のとおりです。

- <Term id="table" />
- <Term id="view" />
- 増分
- 一時
- マテリアライズド・ビュー

dbt では、[カスタム マテリアライゼーション](/guides/create-new-materializations?step=1) を構成することもできます。カスタム マテリアライゼーションは、特定のニーズに合わせて dbt の機能を拡張する強力な手段です。

## マテリアライゼーションの設定
デフォルトでは、dbt モデルは「ビュー」としてマテリアライズされます。以下のタブに示すように、[`materialized` 設定](/reference/resource-configs/materialized) パラメータを指定することで、モデルを異なるマテリアライゼーションで設定できます。
<Tabs>

<TabItem value="Project file">

<File name='dbt_project.yml'>

```yaml
# The following dbt_project.yml configures a project that looks like this:
# .
# └── models
#     ├── csvs
#     │   ├── employees.sql
#     │   └── goals.sql
#     └── events
#         ├── stg_event_log.sql
#         └── stg_event_sessions.sql

name: my_project
version: 1.0.0
config-version: 2

models:
  my_project:
    events:
      # materialize all models in models/events as tables
      +materialized: table
    csvs:
      # this is redundant, and does not need to be set
      +materialized: view
```

</File>

</TabItem>

<TabItem value="Model file">

あるいは、マテリアライゼーションはモデルのSQLファイル内で直接設定することもできます。これは、特定のモデル（例えば、[Redshift固有の設定](/reference/resource-configs/redshift-configs)や[BigQuery固有の設定](/reference/resource-configs/bigquery-configs)）に対して[パフォーマンス最適化]設定も設定している場合に便利です。

<File name='models/events/stg_event_log.sql'>

```sql

{{ config(materialized='table', sort='timestamp', dist='user_id') }}

select *
from ...
```

</File>

</TabItem>

<TabItem value="Property file">

マテリアライゼーションは、モデルの `properties.yml` ファイルでも設定できます。次の例は、`table` マテリアライゼーションタイプを示しています。マテリアライゼーションタイプの完全なリストについては、[マテリアライゼーション](/docs/build/materializations#materializations) を参照してください。

<File name='models/properties.yml'>

```yaml
version: 2

models:
  - name: events
    config:
      materialized: table
```

</File>

</TabItem>

</Tabs>

## マテリアライゼーション


### View
`view` マテリアライゼーションを使用する場合、モデルは実行ごとに `create view as` ステートメントによってビューとして再構築されます。
* **利点:** 追加データは保存されず、ソースデータの上に重ねられたビューには常に最新のレコードが含まれます。
* **欠点:** 大幅な変換を実行するビュー、または他のビューの上に重ねられたビューは、クエリの実行速度が低下します。
* **アドバイス:**
  * 通常は、モデルをビューから開始し、パフォーマンスの問題に気付いた場合にのみ、別のマテリアライゼーションに変更してください。
  * ビューは、列の名前変更や再キャストなどの大幅な変換を行わないモデルに最適です。

### Table
`table` マテリアライゼーションを使用する場合、モデルは実行ごとに `create table as` ステートメントによって <Term id="table" /> として再構築されます。
* **利点:** テーブルへのクエリが高速です。
* **欠点:**
  * 特に複雑な変換の場合、テーブルの再構築に時間がかかることがあります。
  * 基になるソースデータの新しいレコードは、テーブルに自動的に追加されません。
* **アドバイス:**
  * BI ツールによってクエリされるモデルには、エンドユーザーのエクスペリエンスを高速化するために、テーブル マテリアライゼーションを使用してください。
  * また、多くの下流モデルで使用される、処理速度の遅い変換にも、テーブル マテリアライゼーションを使用してください。


### Incremental
`incremental` モデルを使用すると、dbt は前回のモデル実行以降にテーブルにレコードを挿入または更新できます。
* **利点:** 新しいレコードを変換するだけでビルド時間を大幅に短縮できます。
* **欠点:** 増分モデルは追加の設定が必要であり、dbt の高度な使用法となります。増分モデルの使用の詳細については、[こちら](/docs/build/incremental-models) をご覧ください。
* **アドバイス:**
  * 増分モデルはイベント形式のデータに最適です。
  * `dbt の実行速度が遅くなりすぎている場合は、増分モデルを使用してください (つまり、増分モデルから始めないでください)。

### Ephemeral
`ephemeral` モデルはデータベースに直接組み込まれるわけではありません。代わりに、dbt は共通テーブル式 (<Term id="cte" />) を使用して、エフェメラルモデルのコードを依存モデルに補間します。この CTE の識別子は [モデルエイリアス](/docs/build/custom-aliases) を使用して制御できますが、dbt は常にモデル識別子の先頭に `__dbt__cte__` を付加します。

* **利点:**
  * 再利用可能なロジックを記述できます
  - エフェメラルモデルを使用すると、煩雑さが軽減され、<Term id="data-warehouse" /> を整理した状態に保つことができます ([カスタムスキーマ](/docs/build/custom-schemas) を使用してモデルを複数のスキーマに分割することも検討してください)。
* **欠点:**
  * このモデルから直接選択することはできません。
  * [オペレーション](/docs/build/hooks-operations#about-operations) (たとえば、[`dbt run-operation`](/reference/commands/run-operation) を使用して呼び出されるマクロは、エフェメラルノードを `ref()` できません)
  * エフェメラルマテリアライゼーションを過度に使用すると、クエリのデバッグが困難になることもあります。
  * エフェメラルマテリアライゼーションは、[モデルコントラクト](/docs/collaborate/govern/model-contracts#where-are-contracts-supported) をサポートしていません。
* **アドバイス:** エフェメラルマテリアライゼーションは次の場合に使用します。
  * DAG の初期段階にある非常に軽量な変換
  * 1 つまたは 2 つの下流モデルでのみ使用され、
  * 直接クエリする必要がない変換

### Materialized View

`materialized_view` マテリアライズドにより、ターゲット データベースにマテリアライズド ビューを作成および管理できます。
マテリアライズド ビューはビューとテーブルを組み合わせたもので、増分モデルと同様のユースケースで利用できます。

* **メリット:**
  * マテリアライズド ビューは、テーブルのクエリ パフォーマンスとビューのデータの鮮度を組み合わせます。
  * マテリアライズド ビューは増分マテリアライズドとほぼ同様に動作しますが、通常は手動による介入なしに定期的に更新できます (データベースによって異なります)。そのため、増分マテリアライズドで必要な定期的な dbt バッチ更新は不要です。
  * マテリアライズド ビューに対する `dbt run` は、ビューと同様にコード デプロイメントに相当します。
* **デメリット:**
  * マテリアライズド ビューはより複雑なデータベース オブジェクトであるため、データベース プラットフォームでは利用できる構成オプションが少ない傾向があります。詳細については、データベースプラットフォームのドキュメントをご覧ください。
  * マテリアライズドビューは、すべてのデータベースプラットフォームでサポートされているとは限りません。
* **アドバイス:**
  * 増分モデルで十分であるものの、増分ロジックと更新をデータプラットフォームで管理したいユースケースでは、マテリアライズドビューの使用を検討してください。

#### 構成変更の監視

このマテリアライズは、[`on_configuration_change`](/reference/resource-configs/on_configuration_change) 構成を利用します。これは、同名のデータベースオブジェクトの増分的な性質と一致しています。この設定により、dbt は、更新された構成を実装するためにオブジェクトを完全に再作成するのではなく、可能な場合はオブジェクトに直接構成変更を適用しようとします。`dbt-postgres` を例にすると、マテリアライズドビュー自体を再作成することなく、マテリアライズドビューのインデックスを削除および作成できます。

#### スケジュールされた更新

`dbt run` コマンドのコンテキストでは、マテリアライズド・ビューはビューと同様に考えられます。
たとえば、`dbt run` コマンドは、設定または SQL に変更の可能性がある場合にのみ必要です。これは実質的にはデプロイアクションです。
一方、`dbt run` コマンドは、同じシナリオでテーブルに対して実行され、かつテーブル内のデータを更新する必要がある場合にも必要です。
これは、テーブルを基盤とする増分モデルやスナップショットモデルにも当てはまります。
テーブルの場合、スケジュールメカニズムは dbt Cloud またはローカルスケジューラのいずれかです。
テーブルの背後にあるデータを自動的に更新する組み込み機能は用意されていません。
ただし、ほとんどのプラットフォーム（Postgres を除く）では、マテリアライズド・ビューの自動更新を設定する機能が提供されています。
したがって、マテリアライズド・ビューは増分モデルと同様に動作し、データを更新するために dbt を実行する必要がないという利点があります。
もちろん、これは、自動更新がオンになっていて、モデル内で構成されていることを前提としています。

:::info
`dbt-snowflake` はマテリアライズド・ビューをサポートしていません。代わりに動的テーブルを使用します。詳細については、[Snowflake 固有の設定](/reference/resource-configs/snowflake-configs#dynamic-tables) を参照してください。
:::

## Python のマテリアライゼーション

Python モデルは、以下の 2 つのマテリアライゼーションをサポートしています。
- `table`
- `incremental`

増分 Python モデルは、SQL モデルと同じ [増分戦略](/docs/build/incremental-strategy) をすべてサポートします。サポートされる具体的な戦略は、アダプタによって異なります。

Python モデルは `view` または `ephemeral` としてマテリアライゼーションすることはできません。Python は、モデル以外のリソースタイプ（テストやスナップショットなど）ではサポートされていません。

SQL モデルなどの増分モデルでは、入力テーブルを新しいデータ行のみにフィルタリングする必要があります。

<WHCode>

<div warehouse="Snowpark">

<File name='models/my_python_model.py'>

```python
import snowflake.snowpark.functions as F

def model(dbt, session):
    dbt.config(materialized = "incremental")
    df = dbt.ref("upstream_table")

    if dbt.is_incremental:

        # only new rows compared to max in current table
        max_from_this = f"select max(updated_at) from {dbt.this}"
        df = df.filter(df.updated_at >= session.sql(max_from_this).collect()[0][0])

        # or only rows from the past 3 days
        df = df.filter(df.updated_at >= F.dateadd("day", F.lit(-3), F.current_timestamp()))

    ...

    return df
```

</File>

</div>

<div warehouse="PySpark">

<File name='models/my_python_model.py'>

```python
import pyspark.sql.functions as F

def model(dbt, session):
    dbt.config(materialized = "incremental")
    df = dbt.ref("upstream_table")

    if dbt.is_incremental:

        # only new rows compared to max in current table
        max_from_this = f"select max(updated_at) from {dbt.this}"
        df = df.filter(df.updated_at >= session.sql(max_from_this).collect()[0][0])

        # or only rows from the past 3 days
        df = df.filter(df.updated_at >= F.date_add(F.current_timestamp(), F.lit(-3)))

    ...

    return df
```

</File>

</div>

</WHCode>

**注:** BigQuery/Dataproc では、増分モデルは「merge」増分戦略でサポートされています。「insert_overwrite」戦略はまだサポートされていません。

<Snippet path="discourse-help-feed-header" />
<DiscourseHelpFeed tags="materialization"/>

