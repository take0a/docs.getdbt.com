---
title: MetricFlow タイムスパイン
id: metricflow-time-spine
description: "MetricFlow expects a default time spine table called metricflow_time_spine"
sidebar_label: "MetricFlow time spine"
tags: [Metrics, Semantic Layer]
---
<VersionBlock firstVersion="1.9">


分析エンジニアリングでは、さまざまな種類の時間ベースの結合や集計のベーステーブルとして、日付ディメンションまたは「タイムスパイン」テーブルを使用するのが一般的です。
このテーブルの構造は通常、日次または時単位の日付をベースとする列で構成され、会計四半期などの他の時間粒度はベース列に基づいて定義されます。
他のテーブルをベース列のタイムスパインに結合して、ある時点での収益などの指標を計算したり、特定の時間粒度で集計したりできます。

MetricFlow を時間ベースの指標とディメンションで使用するには、タイムスパイン（時間軸）を提供する必要があります。このテーブルは、時間ベースの結合と集計の基盤として機能します。次のいずれかの方法で実行できます:

- タイムスパインをゼロから作成する（例については[タイムスパインの例](#example-time-spine-tables)セクションをご覧ください）、または
- プロジェクト内の既存のテーブル（`dim_date` テーブルなど）を使用する

タイム スパインを取得したら、MetricFlow にその使用方法を指示するために、YAML で設定する必要があります。

## 前提条件
MetricFlow では、タイムスパインを提供する dbt モデルを少なくとも 1 つ定義し、時間ベースの結合に使用する列を（YAML で）指定する必要があります。
つまり、以下の作業が必要です。

- メトリクスに必要な粒度（日次、時間別など）で、少なくとも 1 つの [タイムスパイン](#example-time-spine-tables) を定義します。
必要に応じて、より粗い粒度（月次、年次など）のテーブルを追加定義することもできます。
- [YAML ファイルで各タイムスパインを設定](#configuring-time-spine-in-yaml) して、MetricFlow が列をどのように認識して使用するかを定義します。

重複するタイムスパインは使用できませんのでご注意ください。

MetricFlow は、以下の種類のメトリックとディメンションについて、タイムスパインモデルに対して結合を行います。

- [累積メトリック](/docs/build/cumulative)
- [メトリックオフセット](/docs/build/derived#derived-metric-offset)
- [コンバージョンメトリック](/docs/build/conversion)
- [緩やかに変化するディメンション](/docs/build/dimensions#scd-type-ii)
- [メトリック](/docs/build/metrics-overview) で、`join_to_timespine` 構成が true に設定されている

時間スパイン結合を使用するメトリックおよびディメンション タイプに対して生成された SQL を確認するには、それぞれのドキュメントを参照するか、セマンティック レイヤーをクエリするときに `compile=true` フラグを追加して、コンパイルされた SQL を返します。

## YAMLでタイムスパインを設定する

:::tip Use our mini guide to create a time spine table
タイム スパイン テーブルを作成する方法に関するクイック スタート ガイドについては、[MetricFlow タイム スパイン ミニ ガイド](/guides/mf-time-spine) をご覧ください。
:::

タイムスパインモデルは、プロパティを定義することで、dbtとMetricFlowに特定の列の使用方法を指示する追加の構成を備えた通常のdbtモデルです。
`models/`ディレクトリに、タイムスパイン用の[`models`キー](/reference/model-properties)を追加します。
プロジェクトに既にカレンダーテーブルまたは日付ディメンションが含まれている場合は、そのテーブルをタイムスパインとして設定できます。
そうでない場合は、[タイムスパインテーブルの例](#example-time-spine-tables)を確認して作成してください。関連するモデルファイルが存在しない場合は作成し、[次のセクション](#creating-a-time-spine-table)で説明されている構成を追加してください。
 
タイムスパインモデルを設定する際の注意事項：

- プロジェクトにタイムスパインSQLテーブルが定義されていることを確認してください。
- 説明やテストを追加するのと同じように、その[モデルのプロパティ](/reference/model-properties)の`time_spine`キーの下に設定を追加します。
- セマンティックレイヤーが認識するタイムスパインモデルのみを設定する必要があります。
- 少なくとも、日単位の粒度のタイムスパインテーブルを定義してください。
- オプションで、時間単位など、異なる粒度のタイムスパインテーブルを追加定義することもできます。作成するテーブルを決定する際は、[粒度に関する考慮事項](#granularity-considerations)を確認してください。
- MetricFlowが基になる列を必要な粒度に変換できるように、時間ディメンションの粒度を指定する場合は、[時間粒度に関するドキュメント](/docs/build/dimensions?dimension=time_gran)を参照してください。

:::tip
- 以前に `metricflow_time_spine.sql` モデルを使用していた場合は、YAML で `time_spine` プロパティを設定した後、このモデルを削除できます。
セマンティックレイヤーは新しい設定を自動的に認識します。追加の `.yml` ファイルは必要ありません。
- セマンティックレイヤーの `model` 設定を更新することで、プロジェクトに既に存在する日付ディメンションまたはタイムスパインテーブルを使用するように MetricFlow を構成することもできます。
- 日付ディメンションテーブルがない場合は、[次のセクション](#creating-a-time-spine-table) のコードスニペットを使用してタイムスパインモデルを構築することで作成できます。
:::

### タイムスパインテーブルの作成

MetricFlow は、ミリ秒から年単位までの粒度をサポートしています。
サポートされている粒度の完全なリストについては、[ディメンションページ](/docs/build/dimensions?dimension=time_gran#time) (time_granurity タブ) を参照してください。

タイムスパインテーブルを最初から作成するには、次のコードを dbt プロジェクトに追加します。
この例では、`time_spine_hourly` と `time_spine_daily` という、時間単位と日単位のタイムスパインを作成します。

<VersionBlock firstVersion="1.9">
<File name="models/_models.yml">
  
```yaml
[models:](/reference/model-properties) 
# Hourly time spine
  - name: time_spine_hourly 
    description: my favorite time spine
    time_spine:
      standard_granularity_column: date_hour # column for the standard grain of your table, must be date time type.
      custom_granularities:
        - name: fiscal_year
          column_name: fiscal_year_column
    columns:
      - name: date_hour
        granularity: hour # set granularity at column-level for standard_granularity_column

# Daily time spine
  - name: time_spine_daily
    time_spine:
      standard_granularity_column: date_day # column for the standard grain of your table
    columns:
      - name: date_day
        granularity: day # set granularity at column-level for standard_granularity_column
```
</File>
</VersionBlock>

<!--
<VersionBlock lastVersion="1.8">
<File name="models/_models.yml">
  
```yaml
models:
  - name: time_spine_hourly
    description: A date spine with one row per hour, ranging from 2020-01-01 to 2039-12-31.
    time_spine:
      standard_granularity_column: date_hour # column for the standard grain of your table
    columns:
      - name: date_hour
        granularity: hour # set granularity at column-level for standard_granularity_column
  
  - name: time_spine_daily
    description: A date spine with one row per day, ranging from 2020-01-01 to 2039-12-31.
    time_spine:
      standard_granularity_column: date_day # column for the standard grain of your table
    columns:
      - name: date_day
        granularity: day # set granularity at column-level for standard_granularity_column
```

</File>
</VersionBlock>
-->

- この設定例では、`time_spine_hourly` と `time_spine_daily` というタイムスパインモデルを示しています。`time_spine` キーの下にタイムスパイン設定を設定します。
- `standard_granurity_column` は、[標準粒度](/docs/build/dimensions?dimension=time_gran) のいずれかにマッピングされる列です。この列は `columns` キーの下に設定し、同じモデルで定義されているカスタム粒度列と同等かそれ以上の粒度にする必要があります。
- `columns` キーの下に定義された列（この場合はそれぞれ `date_hour` と `date_day`）を参照する必要があります。
- `granularity` キー（この場合はそれぞれ `hour` と `day`）を使用して、列レベルの粒度を設定します。
- MetricFlow は、タイムスパインテーブルを別のソーステーブルに結合する際に、`standard_granularity_column` を結合キーとして使用します。
- [`custom_granularities` フィールド](#custom-calendar) (dbt Cloud 最新版および dbt Core v1.9 以降で利用可能) を使用すると、組織で使用できる `fiscal_year` や `retail_month` などの非標準期間を指定できます。

サンプルプロジェクトについては、[Jaffle ショップ](https://github.com/dbt-labs/jaffle-sl-template/blob/main/models/marts/_models.yml) の例を参照してください。

### SQL から YAML への移行
プロジェクトに既にタイムスパイン (`metricflow_time_spine.sql`) が含まれている場合は、その設定を YAML に移行することで、非推奨の警告に対処できます。

1. `models/` ディレクトリ内のタイムスパインに対して [`models` キー](/reference/model-properties) を使用して、新規または既存の YAML ファイルに以下の設定を追加します。YAML ファイルには任意の名前を付けます (例: `util/_models.yml`)。

  <File name="models/_models.yml">

  ```yaml
  models:
    - name: all_days
      description: A time spine with one row per day, ranging from 2020-01-01 to 2039-12-31.
      time_spine:
        standard_granularity_column: date_day  # Column for the standard grain of your table
      columns:
        - name: date_day
          granularity: day  # Set the granularity of the column
  ```
  </File>

2. YAML 設定を追加したら、問題を回避するために、既存の `metricflow_time_spine.sql` ファイルをプロジェクトから削除します。

3. 設定をテストし、本番環境のジョブとの互換性を確認します。

`metricflow_time_spine.sql` ファイルから移行する場合は、次の点に注意してください。

- 前の例に示すように、YAML に `time_spine` プロパティを追加して、機能を置き換えます。
- 設定が完了すると、MetricFlow は YAML 設定を認識するため、SQL モデルファイルを安全に削除できます。

### 作成する粒度を選択する際の考慮事項 {#granularity-considerations}

- MetricFlow は、特定のクエリに対して互換性のある最大の粒度を持つタイムスパインを使用することで、可能な限り効率的なクエリを実現します。
たとえば、月単位の粒度のタイムスパインがあり、月単位の粒度でディメンションをクエリする場合、MetricFlow は月単位のタイムスパインを使用します。
日単位のタイムスパインしかない場合、MetricFlow は日単位のタイムスパインを使用し、date_trunc を月に変換します。
- クエリの効率が設定時間やストレージ制約よりも重要な場合は、使用する粒度ごとにタイムスパインを追加できます。
ほとんどのエンジンでは、クエリのパフォーマンスの違いは最小限に抑えられるため、クエリ時にタイムスパインをより粗い粒度に変換しても、クエリに大きなオーバーヘッドは発生しません。
- 予期しないエラーを回避するために、どのディメンションでも最も細かい粒度のタイムスパインを使用することをお勧めします。
たとえば、時間単位の粒度のディメンションがある場合は、時間単位の粒度でタイムスパインを使用する必要があります。

## タイムスパインテーブルの例

以下の例は、異なる粒度でタイムスパインテーブルを作成する方法を示しています:

<!-- no toc -->
- [Seconds](#seconds)
- [Minutes](#minutes)
- [Daily](#daily)
- [Daily (BigQuery)](#daily-bigquery)
- [Hourly](#hourly)

### Seconds

<File name="metricflow_time_spine.sql">

```sql
{{ config(materialized='table') }}

with seconds as (

    {{
        dbt.date_spine(
            'second',
            "date_trunc('second', dateadd(second, -10, current_timestamp()))",
            "date_trunc('second', current_timestamp())"
        )
    }}

),

final as (
    select cast(date_second as timestamp) as second_timestamp
    from seconds
)

select * from final

```
</File>

### Minutes

<File name="metricflow_time_spine.sql">

```sql
{{ config(materialized='table') }}

with minutes as (

    {{
        dbt.date_spine(
            'minute',
            "date_trunc('minute', dateadd(minute, -5, current_timestamp()))",
            "date_trunc('minute', current_timestamp())"
        )
    }}

),

final as (
    select cast(date_minute as timestamp) as minute_timestamp
    from minutes
)

select * from final
```

</File>

### Daily

<File name="metricflow_time_spine.sql">

```sql
{{
    config(
        materialized = 'table',
    )
}}

with days as (

    {{
        dbt.date_spine(
            'day',
            "to_date('01/01/2000','mm/dd/yyyy')",
            "to_date('01/01/2025','mm/dd/yyyy')"
        )
    }}

),

final as (
    select cast(date_day as date) as date_day
    from days
)

select * from final
where date_day > dateadd(year, -4, current_timestamp()) 
and date_day < dateadd(day, 30, current_timestamp())
```

### Daily (BigQuery)

BigQuery を使用している場合は、このモデルを使用してください。BigQuery は `TO_DATE()` ではなく `DATE()` をサポートしています。

<File name="metricflow_time_spine.sql">

```sql

{{config(materialized='table')}}
with days as (
    {{dbt.date_spine(
        'day',
        "DATE(2000,01,01)",
        "DATE(2025,01,01)"
    )
    }}
),

final as (
    select cast(date_day as date) as date_day
    from days
)

select *
from final
-- filter the time spine to a specific range
where date_day > date_add(DATE(current_timestamp()), INTERVAL -4 YEAR)
and date_day < date_add(DATE(current_timestamp()), INTERVAL 30 DAY)
```

</File>

</File>

### Hourly

<File name='time_spine_hourly.sql'>

```sql
{{
    config(
        materialized = 'table',
    )
}}

with hours as (

    {{
        dbt.date_spine(
            'hour',
            "to_date('01/01/2000','mm/dd/yyyy')",
            "to_date('01/01/2025','mm/dd/yyyy')"
        )
    }}

),

final as (
    select cast(date_hour as timestamp) as date_hour
    from hours
)

select * from final
-- filter the time spine to a specific range
where date_day > dateadd(year, -4, current_timestamp()) 
and date_hour < dateadd(day, 30, current_timestamp())
```

</File>


</VersionBlock>

<VersionBlock lastVersion="1.8">

<!-- this whole section is for 1.8 and and lower -->

MetricFlow uses a time spine table to construct cumulative metrics. By default, MetricFlow expects the time spine table to be named `metricflow_time_spine` and doesn't support using a different name. For supported granularities, refer to the [dimensions](/docs/build/dimensions?dimension=time_gran#time) page.

To create this table, you need to create a model in your dbt project called `metricflow_time_spine` and add the following code:

### Daily

<File name='metricflow_time_spine.sql'>


```sql
{{
    config(
        materialized = 'table',
    )
}}

with days as (

    {{
        dbt.date_spine(
            'day',
            "to_date('01/01/2000','mm/dd/yyyy')",
            "to_date('01/01/2025','mm/dd/yyyy')"
        )
    }}

),

final as (
    select cast(date_day as date) as date_day
    from days
)

select * from final
where date_day > dateadd(year, -4, current_timestamp()) 
and date_day  < dateadd(day, 30, current_timestamp())
```

</File>

### Daily (BigQuery)

Use this model if you're using BigQuery. BigQuery supports `DATE()` instead of `TO_DATE()`:

<File name="metricflow_time_spine.sql">

```sql
{{config(materialized='table')}}
with days as (
    {{dbt.date_spine(
        'day',
        "DATE(2000,01,01)",
        "DATE(2025,01,01)"
    )
    }}
),

final as (
    select cast(date_day as date) as date_day
    from days
)

select *
from final
-- filter the time spine to a specific range
where date_day > dateadd(year, -4, current_timestamp()) 
and date_day < dateadd(day, 30, current_timestamp())
```

</File>

You only need to include the `date_day` column in the table. MetricFlow can handle broader levels of detail, but finer grains are only supported in versions 1.9 and higher.

</VersionBlock>


## カスタムカレンダー <Lifecycle status="Preview"/>

:::tip
まずは、[タイム スパイン テーブルの作成方法](/guides/mf-time-spine)に関するミニ ガイドをご覧ください。
:::


<VersionBlock lastVersion="1.8">

The ability to configure custom calendars, such as a fiscal calendar, is available now in [the "Latest" release track in dbt Cloud](/docs/dbt-versions/cloud-release-tracks), and it will be available in [dbt Core v1.9+](/docs/dbt-versions/core-upgrade/upgrading-to-v1.9). 

</VersionBlock>

<VersionBlock firstVersion="1.9">

カスタム日付変換は複雑になる場合があり、組織には簡単に一般化できない独自のニーズがあることがよくあります。
カスタムカレンダーモデルを作成すると、これらの変換をSQLで定義できるため、MetricFlowのネイティブ変換よりも柔軟性が高まります。
このアプローチにより、カスタム列をMetricFlowの粒度にマッピングし直すことができ、一貫性を保ちながら変換を制御できます。

例えば、組織で会計カレンダーなどのカスタムカレンダーを使用している場合、MetricFlowで日付と時刻の操作を使用してカレンダーを設定できます。

- これは、会計四半期や会計週などのカスタムカレンダーに基づいて指標を計算する場合に便利です。
- `custom_granurities` キーを使用して、`day`、`month`、`year` などの標準オプションの代わりに、`retail_month` や `fiscal_week` などの非標準の期間を指定してデータをクエリできます。
- この機能により、時間ベースの指標の計算方法をより細かく制御できます。

<Expandable alt_header="データ型とタイムゾーンの考慮事項">
 
MetricFlow でカスタムカレンダーを使用する場合は、以下の点に注意してください:

- データ型の一貫性 - 正確な比較を行うには、ディメンション列とタイムスパイン列の両方で同じデータ型を使用する必要があります。「DATE_TRUNC」などの関数は、一部のデータベース（Snowflake など）では入力のデータ型を変更しません。異なるデータ型を使用すると、不一致が発生し、結果が不正確になる可能性があります。

  時間ディメンションとタイムスパインには、あらゆる粒度をサポートしている「DATETIME」または「TIMESTAMP」データ型を使用することをお勧めします。「DATE」データ型は、時間や分などのより細かい粒度をサポートしていない場合があります。

- タイムゾーン - 現在、MetricFlow はタイムゾーンの操作を実行しません。タイムゾーン対応データを扱う場合、タイムゾーンが一致していないと、集計や比較の際に予期しない結果が生じる可能性があります。

例えば、タイムスパイン列が `TIMESTAMP` 型で、ディメンション列が `DATE` 型の場合、これらの列の比較は意図したとおりに機能しない可能性があります。この問題を解決するには、`DATE` 列を `TIMESTAMP` 型に変換するか、両方の列のデータ型を同じにしてください。

</Expandable>

### カスタム粒度の追加

カスタム粒度を追加するために、セマンティックレイヤーはカスタムカレンダー設定をサポートしています。これにより、ユーザーは　`fiscal_year` や `retail_month` といった標準以外の期間を使用してデータをクエリできます。
これらのカスタム粒度（すべて小文字）は、モデルのYAML構成を次のように変更することで定義できます。

<File name="models/_models.yml">

```yaml
models:
 - name: my_time_spine
   description: my favorite time spine
   time_spine:
      standard_granularity_column: date_day
      custom_granularities:
        - name: fiscal_year
          column_name: fiscal_year_column
```
</File>

#### 近日公開予定
オフセット計算や前期比計算などの機能も近日中にサポートされる予定です。

</VersionBlock>


## 関連ドキュメント

- [MetricFlow の時間粒度](/docs/build/dimensions?dimension=time_gran#time)
- [MetricFlow 時間スパイン ミニガイド](/guides/mf-time-spine)
