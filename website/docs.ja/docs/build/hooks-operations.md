---
title: "フックと操作"
description: "フックと操作を使用して dbt ワークフローをカスタマイズします。"
id: "hooks-operations"
---

import OnRunCommands from '/snippets/_onrunstart-onrunend-commands.md';

## 関連ドキュメント
* [pre-hook と post-hook](/reference/resource-configs/pre-hook-post-hook)
* [on-run-start と on-run-end](/reference/project-configs/on-run-start-on-run-end)
* [`run-operation` コマンド](/reference/commands/run-operation)

### 前提知識
* [プロジェクト構成](/reference/dbt_project.yml.md)
* [モデル構成](/reference/model-configs)
* [マクロ](/docs/build/jinja-macros#macros)

## フックとオペレーションの使い方

効果的なデータベース管理には、追加のSQL文の実行が必要になる場合があります。例えば、次のようなものがあります。
- UDFの作成
- 行レベルまたは列レベルの権限の管理
- Redshift上のテーブルのバキューム処理
- Redshift Spectrum外部テーブルへのパーティションの作成
- Snowflake上のウェアハウスの再開/一時停止/サイズ変更
- Snowflake上のパイプの更新
- Snowflake上での共有の作成
- Snowflake上のデータベースのクローン作成

dbtはフックとオペレーションを提供しており、これらのステートメントをdbtプロジェクトの一部としてバージョン管理および実行できます。

## フックについて

フックとは、異なるタイミングで実行されるSQL文のスニペットです。
* `pre-hook`: モデル、シード、またはスナップショットが構築される _前_ に実行されます。
* `post-hook`: モデル、シード、またはスナップショットが構築される _後_ に実行されます。
* `on-run-start`: <OnRunCommands/> の _開始_ 時に実行されます。
* `on-run-end`: <OnRunCommands/> の _終了_ 時に実行されます。

フックは、dbt の標準のマテリアライゼーションと設定ですぐに使用できる機能を超えて、カスタムSQLを実行したり、データベース固有のアクションを活用できるようにする、より高度な機能です。

[`grants` リソース構成](/reference/resource-configs/grants) を利用できない場合に限り、`post-hook` を使用してより高度なワークフローを実行できます。

* より複雑な方法で `grants` を適用する必要があるが、dbt Core の `grants` 構成では（まだ）サポートされていない。
* dbt が標準ではサポートしていない後処理を実行する必要がある場合。たとえば、`analyze table`、`alter table set property`、`alter table ... add row access policy` など。

### フックの使用例

フックを使用すると、オペレーションの実行中、またはモデル、シード、スナップショットの作成中に、特定のタイミングでアクションをトリガーできます。

フックをトリガーできるタイミングの詳細については、[`on-run-start` フックと `on-run-end` フック](/reference/project-configs/on-run-start-on-run-end)、および [`pre-hook` フックと `post-hook` フック](/reference/resource-configs/pre-hook-post-hook) のリファレンスセクションを参照してください。

フックを使用すると、dbt ではすぐに使用できないデータベース固有の機能を提供できます。たとえば、`config` ブロックを使用して、`post-hook` を使用して個々のモデルを作成した直後に `ALTER TABLE` ステートメントを実行できます:

<File name='models/<model_name>.sql'>

```sql
{{ config(
    post_hook=[
      "alter table {{ this }} ..."
    ]
) }}
```

</File>


### フック内でのマクロの呼び出し

[マクロ](/docs/build/jinja-macros#macros)を使用してフックロジックをまとめることもできます。[on-run-startフックとon-run-endフック](/reference/project-configs/on-run-start-on-run-end)、および[pre-hookとpost-hook](/reference/resource-configs/pre-hook-post-hook)のリファレンスセクションにある例をいくつかご確認ください。

<File name='models/<model_name>.sql'>

```sql
{{ config(
    pre_hook=[
      "{{ some_macro() }}"
    ]
) }}
```

</File>

<File name='models/properties.yml'>

```yaml
models:
  - name: <model_name>
    config:
      pre_hook:
        - "{{ some_macro() }}"
```

</File>

<File name='dbt_project.yml'>

```yaml
models:
  <project_name>:
    +pre-hook:
      - "{{ some_macro() }}"
```

</File>

## オペレーションについて

オペレーションとは、[`run-operation`](/reference/commands/run-operation) コマンドを使って実行できる [マクロ](/docs/build/jinja-macros#macros) のことです。オペレーションは実際には dbt プロジェクト内の独立したリソースではなく、モデルを実行せずにマクロを呼び出すための便利な手段です。

:::info 操作内でSQLを明示的に実行する
フックとは異なり、マクロ内でクエリを明示的に実行する必要があります。そのためには、[ステートメントブロック](/reference/dbt-jinja-functions/statement-blocks)または[run_query](/reference/dbt-jinja-functions/run_query)マクロなどのヘルパーマクロを使用します。そうでない場合、dbtはクエリを実行せずに文字列として返します。
:::

このマクロは上記のフックと同様のアクションを実行します:

<File name='macros/grant_select.sql'>

```sql
{% macro grant_select(role) %}
{% set sql %}
    grant usage on schema {{ target.schema }} to role {{ role }};
    grant select on all tables in schema {{ target.schema }} to role {{ role }};
    grant select on all views in schema {{ target.schema }} to role {{ role }};
{% endset %}

{% do run_query(sql) %}
{% do log("Privileges granted", info=True) %}
{% endmacro %}

```

</File>

このマクロを操作として呼び出すには、`dbt run-operation grant_select --args '{role: reporter}'` を実行します。

```
$ dbt run-operation grant_select --args '{role: reporter}'
Running with dbt=1.6.0
Privileges granted

```

`run-operation` コマンドの完全な使用方法については、[こちら](/reference/commands/run-operation) をご覧ください。


## 追加例

コミュニティからのこれらの例は、フックと操作のユースケースの一部を示しています。

* [フックとオペレーションを使用した権限付与の詳細な説明（dbt Core バージョン 1.2 より前）](https://discourse.getdbt.com/t/the-exact-grant-statements-we-use-in-a-dbt-project/430)
* [外部テーブルのステージング](https://github.com/dbt-labs/dbt-external-tables)
* [Snowflake でゼロコピークローンを実行して開発環境をリセットする](https://discourse.getdbt.com/t/creating-a-dev-environment-quickly-on-snowflake/1151/2)
* [Redshift ウェアハウスで `vacuum` と `analyze` を実行する](https://github.com/dbt-labs/redshift/tree/0.2.3/#redshift_maintenance_operation-source)
* [Snowflake の共有の作成](https://discourse.getdbt.com/t/how-drizly-is-improving-collaboration-with-external-partners-using-dbt-snowflake-shares/1110)
* [Redshift 上の S3 へのファイルのアンロード](https://github.com/dbt-labs/redshift/tree/0.2.3/#unload_table-source)
* [モデルタイミングの監査イベントの作成](https://github.com/dbt-labs/dbt-event-logging)
* [UDF の作成](https://discourse.getdbt.com/t/using-dbt-to-manage-user-defined-functions/18)
