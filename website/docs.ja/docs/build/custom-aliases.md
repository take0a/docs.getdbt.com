---
title: "カスタムエイリアス"
description: "dbt 内のモデルやその他のリソースのデフォルトの命名規則をオーバーライドするには、カスタム エイリアスを構成します。"
id: "custom-aliases"
---

## 概要

dbt がモデルを実行すると、通常はデータベースにリレーション（<Term id="table" /> または <Term id="view" /> ）が作成されます。ただし、[一時モデル](/docs/build/materializations)の場合は、別のモデルで使用するために <Term id="cte" /> が作成されます。デフォルトでは、dbt はモデルのファイル名を、作成するリレーションまたは CTE の識別子として使用します。この識別子は、[`alias`](/reference/resource-configs/alias) モデル設定を使用して上書きできます。

### モデル名にエイリアスを付ける理由

スキーマとテーブルの名前は、事実上、<Term id="data-warehouse" /> の「ユーザーインターフェース」です。適切に命名されたスキーマとテーブルは、データの利用者に明確な指示を与えるのに役立ちます。[カスタムスキーマ](/docs/build/custom-schemas) と組み合わせることで、モデルのエイリアス設定はウェアハウス設計における強力なメカニズムとなります。

モデルを整理するために使用するファイルの命名スキームが、データプラットフォームの識別子要件に抵触する場合もあります。例えば、ピリオド (`.`) を使用してファイルの名前空間を指定したい場合、データプラットフォームの SQL 方言では、ピリオドが識別子内のスキーマ名とテーブル名の区切りとして解釈されるか、CTE 識別子ではピリオドの使用が一切禁止される可能性があります。このような場合、モデルのエイリアス設定を使用することで、データプラットフォームの識別子要件に違反することなく、モデルファイルの命名方法に柔軟性を持たせることができます。

### 使用方法

`alias` 設定を使用すると、データベース内のモデル識別子の名前を変更できます。次の表は、`alias` が指定されている場合と指定されていない場合、また異なるマテリアライゼーションを持つモデルのデータベース識別子の例を示しています。

| Model | Config | Relation Type | Database Identifier |
| ----- | ------ | --------------| ------------------- |
| ga_sessions.sql | \{\{ config(materialization='view') \}\} | <Term id="view" /> | "analytics"."ga_sessions" |
| ga_sessions.sql | \{\{ config(materialization='view', alias='sessions') \}\} | <Term id="view" /> | "analytics"."sessions" |
| ga_sessions.sql | \{\{ config(materialization='ephemeral') \}\} | <Term id="cte" /> | "\__dbt\__cte\__ga_sessions" |
| ga_sessions.sql | \{\{ config(materialization='ephemeral', alias='sessions') \}\} | <Term id="cte" /> | "\__dbt\__cte\__sessions" |

モデルのエイリアスを設定するには、モデルの `alias` 設定パラメータに値を指定します。例:

<File name='models/google_analytics/ga_sessions.sql'>

```sql

-- This model will be created in the database with the identifier `sessions`
-- Note that in this example, `alias` is used along with a custom schema
{{ config(alias='sessions', schema='google_analytics') }}

select * from ...
```

</File>

Or in a `schema.yml` file.

<File name='models/google_analytics/schema.yml'>

```yaml
models:
  - name: ga_sessions
    config:
      alias: sessions
```

</File>

上記の `ga_sessions` モデルを別のモデルから参照する場合は、通常どおりモデルの _filename_ を指定した `ref()` 関数を使用します。例:

<File name='models/combined_sessions.sql'>

```sql

-- Use the model's filename in ref's, regardless of any aliasing configs

select * from {{ ref('ga_sessions') }}
union all
select * from {{ ref('snowplow_sessions') }}
```

</File>

### generate_alias_name

モデルに生成されるエイリアスは、`generate_alias_name` というマクロによって制御されます。このマクロは、dbt プロジェクト内でオーバーライドすることで、dbt によるモデルのエイリアス設定方法を変更できます。このマクロは、[generate_schema_name](/docs/build/custom-schemas#advanced-custom-schema-configuration) マクロと同様に動作します。

dbt によるエイリアス名生成をオーバーライドするには、dbt プロジェクト内に `generate_alias_name` というマクロを作成します。`generate_alias_name` マクロは、以下の 2 つの引数を取ります。

1. モデル設定で指定されたカスタムエイリアス
2. カスタムエイリアスが生成されるノード

`generate_alias_name` のデフォルト実装では、指定された `alias` 設定（存在する場合）をモデルエイリアスとして使用し、存在しない場合はモデル名を使用します。この実装は次のようになります。

<File name='get_custom_alias.sql'>

```jinja2
{% macro generate_alias_name(custom_alias_name=none, node=none) -%}

    {%- if custom_alias_name -%}

        {{ custom_alias_name | trim }}

    {%- elif node.version -%}

        {{ return(node.name ~ "_v" ~ (node.version | replace(".", "_"))) }}

    {%- else -%}

        {{ node.name }}

    {%- endif -%}

{%- endmacro %}

```

</File>

import WhitespaceControl from '/snippets/_whitespace-control.md';

<WhitespaceControl/>

### Dispatch マクロ - データベースおよび dbt パッケージの SQL エイリアス管理

マクロ `dispatch` のドキュメントを参照してください: ["パッケージ間で異なるグローバルオーバーライドを管理する"](/reference/dbt-jinja-functions/dispatch#managing-different-global-overrides-across-packages)

### 注意点

#### 曖昧なデータベース識別子

エイリアスを使用すると、あいまいな識別子を持つモデルが誤って作成される可能性があります。以下の2つのモデルの場合、dbt はデータベース内に完全に同じ名前（つまり「セッション」）を持つ 2 つの <Term id="view">ビュー</Term> を作成しようとします。

<File name='models/snowplow_sessions.sql'>

```sql
{{ config(alias='sessions') }}

select * from ...
```
</File>

<File name='models/sessions.sql'>

```sql
select * from ...
```

</File>

これらのモデルのうち、2番目に実行された方が「勝者」となり、一般的にdbtの出力は期待どおりにはなりません。この失敗モードを回避するために、dbtはモデル名とエイリアスが本質的に曖昧かどうかをチェックします。曖昧な場合は、次のようなエラーメッセージが表示されます:

```
$ dbt compile
Encountered an error:
Compilation Error
  dbt found two resources with the database representation "analytics.sessions".
  dbt cannot create two resources with identical database representations. To fix this,
  change the "schema" or "alias" configuration of one of these resources:
  - model.my_project.snowplow_sessions (models/snowplow_sessions.sql)
  - model.my_project.sessions (models/sessions.sql)
```

これらのモデルが実際に同じデータベース識別子を持つ必要がある場合は、モデルの 1 つに [カスタム スキーマ](/docs/build/custom-schemas) を構成することでこのエラーを回避できます。

#### モデルのバージョン

**関連ドキュメント:**
- [モデルのバージョン](/docs/collaborate/govern/model-versions)
- [`versions`](/reference/resource-properties/versions#alias)

デフォルトでは、dbt はバージョン管理されたモデルを `<model_name>_v<v>` というエイリアスで作成します。`<v>` はバージョンの一意の識別子です。この動作は、バージョン管理されていないモデルと同様に、カスタム `alias` を設定するか、`generate_alias_name` マクロを再実装することでカスタマイズできます。

## 関連ドキュメント

- [dbt モデルのデータベース、スキーマ、エイリアスのカスタマイズ](/guides/customize-schema-alias?step=1) で、dbt モデルのデータベース、スキーマ、エイリアスのカスタマイズ方法をご確認ください。
- [カスタム スキーマ](/docs/build/custom-schemas) で、dbt スキーマのカスタマイズ方法をご確認ください。
- [カスタム データベース](/docs/build/custom-databases) で、dbt データベースのカスタマイズ方法をご確認ください。
