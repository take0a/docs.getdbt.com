---
title: "増分モデルを構成する"
description: "dbt で開発するときに増分モデルを構成および最適化する方法を学習します。"
id: "incremental-models"
keywords: ["incremental models", "incremental materialization","incremental", "materialization", "incremental model", "incremental strategy", "incremental model configuration"]
intro_text: "dbt で開発するときに増分モデルを構成および最適化する方法を学習します。"
---

増分モデルは、<Term id="data-warehouse" /> 内にテーブルとして構築されます。モデルを初めて実行すると、ソースデータのすべての行が変換されて <Term id="table" /> が構築されます。以降の実行では、dbt はフィルター処理するように指定されたソースデータの行のみを変換し、既に構築されているターゲットテーブルに挿入します。

多くの場合、増分実行でフィルター処理する行は、前回の dbt 実行以降に作成または更新されたソースデータの行になります。そのため、dbt を実行するたびに、モデルは増分的に構築されます。

増分モデルを使用すると、変換が必要なデータの量が制限されるため、変換の実行時間が大幅に短縮されます。これにより、ウェアハウスのパフォーマンスが向上し、コンピューティングコストが削減されます。

## 増分マテリアライゼーションを構成する

dbt に組み込まれている他の <Term id="materialization">マテリアライゼーション</Term> と同様に、増分モデルは `select` ステートメントで定義され、マテリアライゼーションは構成ブロックで定義されます。
```sql
{{
    config(
        materialized='incremental'
    )
}}

select ...

```

増分モデルを使用するには、dbt に以下の情報も指定する必要があります:

* 増分実行時に行をフィルタリングする方法
* モデルの一意のキー（存在する場合）

### is_incremental() マクロを理解する

`is_incremental()` マクロは増分マテリアライゼーションを実現します。以下の条件をすべて満たす場合、`True` を返します。

- モデルがデータベースにテーブルとして既に存在していること
- `full-refresh` フラグが渡されていないこと
- 実行中のモデルが `materialized='incremental'` で構成されていること

`is_incremental()` が `True` と評価されても `False` と評価されても、モデル内の SQL が有効である必要があることに注意してください。

### 増分実行時の行のフィルタリング

増分実行時に変換する行を dbt に指示するには、これらの行をフィルタリングする有効な SQL を `is_incremental()` マクロで囲みます。

多くの場合、「新しい」行、つまり dbt がこのモデルを最後に実行してから作成された行をフィルタリングする必要があります。このモデルの最新の実行時のタイムスタンプを確認する最適な方法は、ターゲットテーブルの最新のタイムスタンプを確認することです。dbt では、"[\{\{ this \}\}](/reference/dbt-jinja-functions/this)" 変数を使用して、ターゲットテーブルを簡単にクエリできます。

また、新規レコードと更新レコードの両方を取得したいというニーズもよくあります。更新レコードの場合は、[一意のキーを定義する](#一意のキーを定義する (オプション)) ことで、変更されたレコードが重複として取り込まれないようにする必要があります。 `is_incremental()` コードは、dbt がこのモデルを最後に実行してから作成または変更された行をチェックします。

例えば、計算に時間がかかる列の変換を含むモデルは、次のように増分的に構築できます。

<File name='models/stg_events.sql'>

```sql
{{
    config(
        materialized='incremental'
    )
}}

select
    *,
    my_slow_function(my_column)

from {{ ref('app_data_events') }}

{% if is_incremental() %}

  -- this filter will only be applied on an incremental run
  -- (uses >= to include records whose timestamp occurred since the last run of this model)
  -- (If event_time is NULL or the table is truncated, the condition will always be true and load all records)
where event_time >= (select coalesce(max(event_time),'1900-01-01') from {{ this }} )

{% endif %}
```

</File>



:::tip 増分モデルの最適化

共通テーブル式（CTE）を使用するより複雑な増分モデルでは、`is_incremental()`マクロの位置がクエリパフォーマンスに与える影響を考慮する必要があります。一部のウェアハウスでは、レコードを早期にフィルタリングすることで、クエリの実行時間を大幅に短縮できる場合があります。

:::

### ユニークキーの定義（オプション）

`unique_key` を使用すると、新しい行を追加するだけでなく、既存の行を更新できます。既存の `unique_key` に新しい情報が到着した場合、その新しい情報はテーブルに追加されるのではなく、現在の情報を置き換えることができます。重複する行が到着した場合は無視できます。更新対象を特定の列のみに選択するなど、この更新動作を管理するための詳細なオプションについては、[戦略固有の設定](/docs/build/incremental-strategy#strategy-specific-configs) を参照してください。

`unique_key` を指定しないと、追加のみの動作となり、dbt はモデルの SQL によって返されたすべての行を、行が重複しているかどうかに関係なく、既存のターゲットテーブルに挿入します。

オプションの `unique_key` パラメータは、モデルの粒度を定義するフィールド（またはフィールドの組み合わせ）を指定します。つまり、フィールドは単一の一意の行を識別します。モデルの先頭にある設定ブロックで `unique_key` を定義できます。これは単一の列名または列名のリストで表すことができます。

`unique_key` は、モデル定義において、単一の列を表す文字列、または一重引用符で囲んだ列名のリスト（組み合わせて使用​​可能）として指定する必要があります（例：`['col1', 'col2', …])`）。このように使用される列には null を含めないでください。null があると、増分モデルで行の一致が失敗し、重複行が生成される可能性があります。各列に null がないことを確認するか (たとえば、`coalesce(COLUMN_NAME, 'VALUE_IF_NULL')` を使用)、単一列の [代理キー](https://www.getdbt.com/blog/guide-to-surrogate-key) を定義します (たとえば、[`dbt_utils.generate_surrogate_key`](https://github.com/dbt-labs/dbt-utils#generate_surrogate_key-source) を使用)。

:::tip
各行を一意に識別するために複数の列を組み合わせる必要がある場合は、これらの列を文字列式 (`unique_key = 'concat(user_id, session_number)'`) ではなく、リスト (`unique_key = ['user_id', 'session_number']`) として渡すことをお勧めします。

より汎用性の高い最初の構文を使用することで、dbt はデータベースに適した方法で列を増分モデルの実体化にテンプレート化できます。

この方法でリストを渡す場合は、各列に null が含まれていないことを確認してください。null が含まれていると、増分モデルの実行が失敗する可能性があります。

あるいは、[`dbt_utils.generate_surrogate_key`](https://github.com/dbt-labs/dbt-utils#generate_surrogate_key-source) などを使用して、単一列の [代理キー](https://www.getdbt.com/blog/guide-to-surrogate-key) を定義することもできます。
:::

`unique_key` を定義すると、dbt モデルから返される「新しい」データの各行に対して、次の動作が行われます。

* 「新しい」モデルデータと「古い」モデルデータに同じ `unique_key` が存在する場合、dbt は古い行を新しいデータ行で更新/置換します。更新/置換の具体的な方法は、データベース、[増分戦略](/docs/build/incremental-strategy)、および [戦略固有の設定](/docs/build/incremental-strategy#strategy-specific-configs) によって異なります。
* 「古い」データに `unique_key` が存在しない場合、dbt は行全体をテーブルに挿入します。

既存のターゲットテーブルまたは新しい増分行のいずれかに、unique_key が複数行存在する場合、データベースと[増分戦略](/docs/build/incremental-strategy)によっては増分モデルが失敗する可能性があるのでご注意ください。増分モデルの実行で問題が発生した場合は、既存のデータベーステーブルと新しい増分行の両方で、unique_key が本当に一意であることを再確認することをお勧めします。[代理キーの詳細については、こちら](https://www.getdbt.com/blog/guide-to-surrogate-key)をご覧ください。

:::info
`delete+insert` + `merge` などの一般的な増分戦略では `unique_key` が使用される場合がありますが、他の戦略では使用されません。例えば、`insert_overwrite` 戦略は個々の行ではなくデータのパーティションを処理するため、 `unique_key` を使用しません。詳細については、[incremental_strategy について](/docs/build/incremental-strategy) をご覧ください。
:::

#### `unique_key` の例

イベントストリームに基づいて、1 日あたりのアクティブユーザー数（DAU）を計算するモデルを考えてみましょう。ソースデータが到着したら、dbt が最後に実行された日とそれ以降の日付の両方について、DAU 数を再計算する必要があります。モデルは次のようになります:

<File name='models/staging/fct_daily_active_users.sql'>

```sql
{{
    config(
        materialized='incremental',
        unique_key='date_day'
    )
}}

select
    date_trunc('day', event_at) as date_day,
    count(distinct user_id) as daily_active_users

from {{ ref('app_data_events') }}


{% if is_incremental() %}

  -- this filter will only be applied on an incremental run
  -- (uses >= to include records arriving later on the same day as the last run of this model)
  where date_day >= (select coalesce(max(date_day), '1900-01-01') from {{ this }})

{% endif %}

group by 1
```

</File>

このモデルを `unique_key` パラメータなしで段階的に構築すると、ターゲットテーブルに1日分の行が複数作成され、その日にdbtが実行されるたびに1行ずつ更新されます。代わりに `unique_key` パラメータを含めることで、既存の行が更新されます。

## 増分モデルを再構築するにはどうすればよいですか？
増分モデルのロジックが変更された場合、新しいデータ行の変換が、ターゲットテーブルに保存されている過去の変換と異なる可能性があります。この場合、増分モデルを再構築する必要があります。

dbt に増分モデル全体を最初から再構築させるには、コマンドラインで `--full-refresh` フラグを使用します。このフラグにより​​、dbt はデータベース内の既存のターゲットテーブルを削除してから、完全に再構築します。

```bash
$ dbt run --full-refresh --select my_incremental_model+
```

末尾の `+` で示されているように、下流のモデルも再構築することをお勧めします。

オプションで [`full_refresh config`](/reference/resource-configs/full_refresh) を使用して、プロジェクトレベルまたはリソースレベルでリソースを常にフルリフレッシュするか、または常にフルリフレッシュしないかを設定できます。true または false で指定した場合、`full_refresh` 設定は `--full-refresh` フラグの有無よりも優先されます。

詳細な使用方法については、[dbt run](/reference/commands/run) のドキュメントをご覧ください。

## 増分モデルの列が変更された場合はどうなりますか？

増分モデルでは、オプションの `on_schema_change` パラメータを追加して設定することで、増分モデルの列が変更された場合に制御を強化できます。これらのオプションにより、スキーマが変更されても dbt は増分モデルの実行を継続できるため、`--full-refresh` シナリオが減り、クエリコストが削減されます。

`on_schema_change` 設定は次のように構成できます。

<File name='dbt_project.yml'>

```yaml
models:
  +on_schema_change: "sync_all_columns"
```

</File>

<File name='models/staging/fct_daily_active_users.sql'>

```sql
{{
    config(
        materialized='incremental',
        unique_key='date_day',
        on_schema_change='fail'
    )
}}
```

</File>

`on_schema_change` に指定できる値は次のとおりです。

* `ignore`: デフォルトの動作（下記参照）。
* `fail`: ソーススキーマとターゲットスキーマが異なる場合にエラーメッセージをトリガーします。
* `append_new_columns`: 既存のテーブルに新しい列を追加します。この設定では、新しいデータに存在しない列は既存のテーブルから削除されません。
* `sync_all_columns`: 既存のテーブルに新しい列を追加し、現在欠落している列を削除します。これにはデータ型の変更が*含まれる*ことに注意してください。BigQuery では、列の型を変更するには <Term id="table" /> 全体のスキャンが必要です。実装時にはトレードオフに注意してください。

**注**: `on_schema_change` のいずれの動作も、新しく追加された列の古いレコードの値をバックフィルしません。これらの値を入力する必要がある場合は、手動で更新を実行するか、`--full-refresh` をトリガーすることをお勧めします。

:::caution `on_schema_change` は最上位レベルの変更を追跡します。

現在、`on_schema_change` は最上位レベルの列の変更のみを追跡します。ネストされた列の変更は追跡しません。たとえば、BigQuery では、`on_schema_change` が適切に設定されていても、ネストされた列の追加、削除、または変更によってスキーマ変更がトリガーされることはありません。

:::

### デフォルトの動作

これはデフォルトで設定されている `on_schema_change: ignore` の動作です。

増分モデルに列を追加して `dbt run` を実行すると、その列はターゲットテーブルに表示されません。

増分モデルから列を削除して `dbt run` を実行すると、`dbt run` は失敗します。

代わりに、増分ロジックが変更されるたびに、増分モデルと下流のモデルの両方でフルリフレッシュを実行してください。

<Snippet path="discourse-help-feed-header" />
<DiscourseHelpFeed tags="incremental"/>
