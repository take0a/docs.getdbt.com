---
title: "New concepts in the dbt Fusion engine"
id: "new-concepts"
sidebar_label: "New Concepts"
description: "New concepts and configurations you will encounter when you install the dbt Fusion engine."
pagination_next: null
pagination_prev: null
---

# New concepts <Lifecycle status="beta" />

<IntroText>

Fusion を使用する際に遭遇するまったく新しい概念について学習します。

</IntroText>


import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';

<FusionBeta />

新しい dbt Fusion エンジンは、SQL をコンパイルして静的に分析し、方言を考慮した検証と列レベルの系統抽出を行います。つまり、`dbt compile` を実行すると、モデルを実行する前に、dbt プロジェクト内のすべてのモデルの完全な論理プランを生成して分析します。
この機能を実現するために、Fusion では**2 つの新しい概念** が導入されています。
- **コンパイル戦略** (事前コンパイルまたはジャストインタイム) は、DAG 実行前または実行中にモードをコンパイルするかどうかを決定します。
- dbt モデル内のコード (SQL) の**静的分析**。これを有効にすると、Fusion は論理プランを生成、検証し、モデルのプランを静的に分析して、列レベルの系統やその他の豊富なメタデータを抽出します。

dbt コアでは、`compile` は Jinja のレンダリングを意味します。dbt Fusion では、compile は新しいステップを導入し、SQL を分析します。したがって、Fusion の `compile` は `render` (Jinja) と `analyze` (SQL) の 2 つのステップに分けられます。

## コンパイル戦略

[事前コンパイル](https://en.wikipedia.org/wiki/Ahead-of-time_compilation)は、Fusionのデフォルトのコンパイル戦略です。つまり、ユーザーがFusionで「dbt run」を実行すると、まずすべてのモデルがコンパイルされ、その後実行されます。これにより、パフォーマンスが大幅に向上し、実行前にDAGの正確性が保証されます。これは、Rust、Typescript、Javaなどのコンパイル言語をモデル化しており、コンパイラが実行前にすべての問題を事前に検出します。

<Lightbox src="/img/fusion/aot-compilation.png" title="Ahead-of-time compilation strategy (Fusion default)" />

[ジャストインタイムコンパイル](https://en.wikipedia.org/wiki/Just-in-time_compilation)は<Constant name="core" />のデフォルトの動作です。dbtはDAG内の各モデルを順番にコンパイルし、実行します。これは、上流モデルの結果が下流モデルのテンプレート化に使用されるイントロスペクティブクエリの入力となる場合に必要です。下流モデルは、上流モデルのビルドが完了した後に「ジャストインタイム」でコンパイルする必要があります。

<Lightbox src="/img/fusion/jit-compilation.png" title="Just-In-time compilation strategy (dbt Core)" />

### 動的テンプレート

Fusion はパフォーマンスと検証のメリットを最大化するために、デフォルトで事前コンパイル (AOT) を採用していますが、動的 Jinja テンプレート ([イントロスペクティブクエリ](/faqs/Warehouse/db-connection-dbt-compile)) を使用するモデルでは、ジャストインタイムコンパイルが必要です。

[run_query](/reference/dbt-jinja-functions/run_query) や [`dbt_utils.get_column_values`](https://github.com/dbt-labs/dbt-utils?tab=readme-ov-file#get_column_values-source) など、データプラットフォームの状態に依存する動的テンプレート関数は、生成する SQL を決定するためにデータベースにクエリを実行する必要があります。

これらのパターンは、AOT コンパイルにおいて「鶏が先か卵が先か」という問題を引き起こします。Fusion は、上流モデルを実行してその結果をクエリするまで、生成される SQL を把握できないのです。下流モデルのSQLは、上流モデルで実行時に生成されるデータに依存するため、「ジャストインタイム」でコンパイルする必要があります。

**Fusionは、DAG内の他のすべてのモデルではAOTコンパイルを維持しながら、JITコンパイルを必要とするモデルのみをインテリジェントに切り替えます。** この選択的なアプローチにより、両方のメリットが得られます。

- 動的テンプレートを使用しないモデルは、AOTのパフォーマンス向上と事前検証の恩恵を受けられます。
- 動的テンプレートを使用するモデルは、JITコンパイルでも正常に動作します。
- コンパイルモードは、DAG全体ではなく、モデルごとに決定されます。

例えば、次のようなDAGの場合：
<Lightbox src="/img/fusion/introspection-normal.png" title="Fusion switches to JIT compilation since all model_b contains an introspection query" />


Fusion は以下を実行します。
1. `model_a`、`model_b`、`model_d` を事前にレンダリングします。
2. DAG の順序で、`model_a`、`model_b` の SQL を解析します。
3. `model_a → model_b` を実行します。
4. `model_c` を JIT コンパイル実行に切り替えます（動的テンプレートのため）。
5. SQL 解析を続行し、`model_d` を実行します（動的モデルに依存するため）。

ただし、並列ブランチがある場合は次のようになります。
<Lightbox src="/img/fusion/introspection-mixed.png" title="Fusion switches to JIT compilation _only_ for model_c (dynamic templating) and its downstreams" />


Fusion は以下の処理を実行します。
1. `model_a`、`model_b`、`model_x`、`model_y`、`model_z` はいずれも動的テンプレートモデルに依存しないため、AOT コンパイルされます。また、`model_d` は Jinja が動的ではないため、AOT レンダリングされます。
2. `model_c` を JIT コンパイルに切り替えて実行します。
3. `model_d` を JIT 解析して実行します。

このきめ細かなアプローチにより、動的 SQL パターンとの互換性を維持しながら、最大限のパフォーマンスと信頼性が確保されます。

## `static_analysis` 設定

Fusion は、サポートされている方言の SQL の大部分を静的に解析できますが、いくつかの例外と制限があります。プロジェクト内の特定のモデルにサポートされていない SQL や動的 SQL が含まれている場合、新しい静的 (SQL) 解析機能のオン/オフを切り替えることができます。この設定は下流に伝播されるため、上流モデルの静的解析がオフになっていると、Fusion はそのモデルを参照するすべての下流モデルの静的解析もオフになります。

## 使用方法

`static_analysis` は、モデルレベルの構成（推奨）として設定することも、実行全体に対する CLI フラグとして設定することもできます。後者はモデルレベルの構成をオーバーライドし、デバッグを支援することを目的としています。構成の詳細については、[CLI オプション](/reference/global-configs/command-line-options) および [構成とプロパティ](/reference/configs-and-properties) を参照してください。

`static_analysis` モデルレベル構成では、以下のオプションを使用します。

- `on`: (デフォルト) SQL を静的に解析します。
- `off`: モデルと、それに依存するすべての下流モデルに対する SQL 解析をスキップします。Fusion は、モデルがイントロスペクトクエリを使用して動的にテンプレート化されていることを検出すると、このモデルとすべての下流モデルに対する静的解析を自動的に停止し、ジャストインタイムコンパイルに切り替えます。
- `unsafe`: 動的に生成されたモデルの SQL を Fusion に強制的に解析させます。ユーザーは、データプラットフォームや上流モデルの状態の変化に応じて、コンパイルから実行までの間にSQLが変化するリスクを負うことになります。実行時には、結果のSQLが無効になったり、異なるSQLになったりする可能性があります。Fusionは、このモデルと下流のモデルに対してJITコンパイルに切り替えます。

<File name='models/<filename>.yml'>

```yml
version: 2

models:
  - name: <model_name>
    config:
      static_analysis: unsafe
      ...
```

</File>

CLI フラグは、実行全体にわたって `--static-analysis=unsafe` または `--static-analysis=off` を使用します。これはモデル レベルの構成よりも優先されます：

```bash
dbt run --static-analysis=unsafe
```

## 例：新しい概念の活用例

`model_a → model_b → model_c → model_d` というDAGを想像してみてください。これらのモデルはすべて静的SQLで定義されています。

### デフォルトの動作（`static_analysis: on`）

- `dbt compile` 実行中、Fusion はすべてのモデルをコンパイルして解析します。
- `dbt run` 実行中、Fusion はすべてのモデルをコンパイルして解析し、その後すべてのモデルを実行します。

### 動的SQLの追加後

`model_c` を更新し、動的SQL（`dbt_utils.get_column_values` など）を導入します。Fusion は `model_c` に動的Jinjaが含まれていることを自動的に検出します。`model_c+` の静的解析を無効にし、Just-In-Time コンパイル戦略に切り替えます。

`dbt compile` 実行中、Fusion は以下の処理を行います。
- プロジェクトを解析し、`model_c` が動的であることを検出し、`model_c+` の `static_analysis: off` を設定します。
- `model_a` と `model_b` をコンパイルして解析します。
- `model_c+` (`model_d` を含む) の解析をスキップします。

`dbt run` 実行中、Fusion は以下の処理を行います。
- プロジェクトを解析し、`model_c` が動的であることを検出し、`model_c+` の `static_analysis: off` とジャストインタイムレンダリングを設定します。
- `model_a` と `model_b` をコンパイルして解析します。
- `model_a → model_b` を実行します。
- `model_c` の SQL をレンダリングし (静的解析なし)、`model_c` を実行します。
- `model_d` の SQL をレンダリングし (静的解析なし)、`model_d` を実行します。

### `model_c` に `static_analysis: unsafe` を明示的に設定します。

この設定は、`model_c` が動的テンプレート化されているにもかかわらず、Fusion に静的解析を試行するように指示します。

`dbt compile` 実行中、Fusion は以下の処理を行います。
- プロジェクトを解析し、`model_c` が安全でないことを検出しますが、明示的なユーザー設定の `static_analysis: unsafe` により、`model_c+` の静的解析を無効にしないように Fusion に指示します。
- すべてのモデルをコンパイルして解析します。`model_b` のイントロスペクションクエリでは、以前に構築されたテーブルまたは本番環境（defer を使用している場合）のデータを使用します。

`dbt run` 実行中、Fusion は以下の処理を行います。
- プロジェクトを解析し、`model_c` が安全でないことを検出しますが、明示的なユーザー設定の `static_analysis: unsafe` により、`model_c+` の静的解析を無効にしないように Fusion に指示します。これらのモデルについては、Fusion は引き続き JIT コンパイルに切り替えます。
- `model_a`、`model_b` をコンパイルして解析する
- `model_a → model_b` を実行する
- `model_c` をコンパイル（静的解析を含む）し、`model_c` を実行する
- `model_d` をコンパイル（静的解析を含む）し、`model_d` を実行する

- **警告:** `model_c` および `model_d` に対して実際に実行される SQL は、コンパイル時に解析された内容と異なる場合があり、スキーマの不一致やエラーが発生する可能性があります。これらのエラーの多くは静的解析中に検出されますが、検出は `model_a → model_b` が既にマテリアライズされた後の「ジャストインタイム」で行われます。

## 静的解析の制限事項

Fusion は、Snowflake やその他のデータウェアハウスでカスタム関数を定義するユーザー定義関数 (UDF) をコンパイルできません。モデルで UDF を使用する場合は、`static_analysis: off` を設定する必要があります。将来的には、Fusion 内で UDF の定義とコンパイルのネイティブサポートを追加する予定です。

Fusion は、dbt-jinja 内のイントロスペクトクエリ呼び出しを介して、動的テンプレート SQL を自動的に検出しますが、Snowflake の `PIVOT` 関数などの動的 SQL は検出できません。モデルで `static_analysis: off` を設定するか、モデルをリファクタリングして静的に強制可能なスキーマを作成することができます (以下の例を参照)。

### 動的SQL

現在、SQL機能（Jinjaではなく）に基づいてスキーマが動的であるかどうかを検出できません。例えば、Snowflakeで`ANY`キーワードを使用する場合、次のようになります。

```sql
with quarterly_sales as (
  select * from values
    (1, 10000, '2023_Q1'),
    (1, 400, '2023_Q1'),
    (2, 4500, '2023_Q1'),
    (2, 35000, '2023_Q1'),
    (3, 10200, '2023_Q4')
  as quarterly_sales(emp_id, amount, quarter)
)

select *
from quarterly_sales
pivot (
  sum(amount) for quarter in (ANY)
)
order by emp_id
```

このサンプル モデルでは、SQL 機能 (Jinja ではない) に基づく動的スキーマが使用されているため、エラーが発生します。

```terminal
error: dbt0432: PIVOT ANY is not compilable
  --> models/example/my_first_model.sql:12:29 (target/compiled/models/example/my_first_model.sql:12:29)
```

このエラーを修正するには、次の操作を実行できます。
- モデルを `static_analysis: off` で構成する
- モデルをリファクタリングして、動的な Jinja テンプレート（例: `dbt_utils.get_column_values`）を使用するようにし、モデルを `static_analysis: unsafe` で構成する
- モデルを静的ピボットにリファクタリングして、安全な静的解析のメリットを享受する

```sql
with quarterly_sales as ( 
    select * from values
    (1, 10000, '2023_01'),
    (1, 400, '2023_01'),
    (2, 4500, '2023_01'),
    (2, 35000, '2023_01'),
    (3, 10200, '2023_04')   
as quarterly_sales(emp_id, amount, quarter)
)

select * from quarterly_sales pivot (
sum(amount) for quarter in ('2023_01', '2023_04'))
order by emp_id
```

### UDFs

モデル SQL でユーザー定義関数を呼び出す場合:

```sql
select my_example_udf(player_id)
    from {{ source('raw_data', 'raw_players') }}
```

Fusion は静的解析中にエラーを発生させます:

```terminal
error: dbt0209: No function MY_EXAMPLE_UDF
  --> models/staging/stg_players.sql:7:9 (target/compiled/models/staging/stg_players.sql:7:9)
```

このエラーを修正するには、該当モデルに対して「static_analysis: off」を設定する必要があります。これにより、下流のモデルに対する静的解析も無効になります。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />
