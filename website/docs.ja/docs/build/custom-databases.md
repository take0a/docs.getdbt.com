---
title: "カスタムデータベース"
id: "custom-databases"
---


:::info 命名について

ウェアハウスによって、論理データベースの名称は異なります。このドキュメントでは、Snowflake、Redshift、Postgres では「データベース」、BigQuery では「プロジェクト」、Databricks Unity Catalog では「カタログ」について説明します。

BigQuery プロジェクト構成では、`project` と `database` の値は互換性があります。
:::

## カスタムデータベースの設定

dbt モデルが構築される論理データベースは、`database` モデル設定を使用して設定できます。この設定がモデルに指定されていない場合、dbt は `profiles.yml` ファイルのアクティブターゲットに設定されたデータベースを使用します。`database` 設定がモデルに指定されている場合、dbt は設定されたデータベースにモデルを構築します。

`database` 設定は、`dbt_project.yml` ファイル内のモデルグループに対して、またはモデル SQL ファイル内の個々のモデルに対して指定できます。

### `dbt_project.yml` でデータベースのオーバーライドを設定する:

この設定により、`jaffle_shop` プロジェクト内のすべてのモデルが `jaffle_shop` というデータベースにビルドされるように変更されます。

<File name='dbt_project.yml'>

```yaml
name: jaffle_shop

models:
  jaffle_shop:
    +database: jaffle_shop

    # For BigQuery users:
    # project: jaffle_shop
```

</File>

### モデルファイルでデータベースのオーバーライドを設定する

この設定は、特定のモデルを「jaffle_shop」というデータベースにビルドするように変更します。

<File name='models/my_model.sql'>

```sql

{{ config(database="jaffle_shop") }}

select * from ...
```

</File>

### generate_database_name

モデル用に生成されるデータベース名は、`generate_database_name` というマクロによって制御されます。このマクロを dbt プロジェクト内でオーバーライドすることで、dbt によるモデルデータベース名の生成方法を変更できます。このマクロは、[generate_schema_name](/docs/build/custom-schemas#advanced-custom-schema-configuration) マクロと同様に動作します。

dbt によるデータベース名生成をオーバーライドするには、独自の dbt プロジェクト内に `generate_database_name` というマクロを作成します。`generate_database_name` マクロは、以下の 2 つの引数を受け取ります。

1. モデル構成で指定されたカスタムデータベース
2. カスタムデータベースが生成されるノード

`generate_database_name` のデフォルト実装では、指定された `database` 構成が存在する場合はそれをそのまま使用し、存在しない場合はアクティブな `target` で構成されたデータベースを使用します。この実装は次のようになります。

<File name='get_custom_database.sql'>

```jinja2
{% macro generate_database_name(custom_database_name=none, node=none) -%}

    {%- set default_database = target.database -%}
    {%- if custom_database_name is none -%}

        {{ default_database }}

    {%- else -%}

        {{ custom_database_name | trim }}

    {%- endif -%}

{%- endmacro %}

```

</File>

import WhitespaceControl from '/snippets/_whitespace-control.md';

<WhitespaceControl/>

### パッケージ間で異なる動作を管理する

マクロ `dispatch` のドキュメントを参照してください: ["パッケージ間で異なるグローバルオーバーライドを管理する"](/reference/dbt-jinja-functions/dispatch)

## 考慮事項

### BigQuery

dbt が BigQuery 接続を開く際、アクティブな `profiles.yml` ターゲットで定義された `project_id` が使用されます。一部のモデルが他のプロジェクトでビルドされるように設定されている場合でも、dbt 実行中に実行されるクエリに対しては、この `project_id` に対して課金されます。

## 関連ドキュメント

- [dbt モデルのデータベース、スキーマ、エイリアスのカスタマイズ](/guides/customize-schema-alias?step=1) で、dbt モデルのデータベース、スキーマ、エイリアスのカスタマイズ方法をご確認ください。
- [カスタム スキーマ](/docs/build/custom-schemas) で、dbt モデル スキーマのカスタマイズ方法をご確認ください。
- [カスタム エイリアス](/docs/build/custom-aliases) で、dbt モデルのエイリアス名のカスタマイズ方法をご確認ください。
