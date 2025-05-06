---
title: "カスタムスキーマ"
description: "データベース内の dbt モデルのテーブルとビューのカスタム スキーマを構成します。"
id: "custom-schemas"
pagination_next: "docs/build/custom-databases"
---

デフォルトでは、すべての dbt モデルは、[環境](/docs/dbt-cloud-environments) (dbt Cloud) または [プロファイルのターゲット](/docs/core/dbt-core-environments) (dbt Core) で指定されたスキーマで構築されます。このデフォルトのスキーマは、_ターゲット スキーマ_ と呼ばれます。

多数のモデルを含む dbt プロジェクトでは、複数のスキーマにまたがってモデルを構築し、類似のモデルをグループ化するのが一般的です。たとえば、次のようなことが考えられます。

* モデルを使用する事業部門に基づいてモデルをグループ化し、`core`、`marketing`、`finance`、`support` などのスキーマを作成します。
* 中間モデルを `staging` スキーマで非表示にし、エンドユーザーがクエリする必要があるモデルのみを `analytics` スキーマで表示します。

これを行うには、カスタム スキーマを指定します。dbt は、カスタム スキーマをターゲット スキーマに追加することで、モデルのスキーマ名を生成します。たとえば、`<target_schema>_<custom_schema>` です。

| Target schema | Custom schema | Resulting schema |
| ------------- | ------------- | ---------------- |
| analytics_prod | None | analytics_prod |
| alice_dev | None | alice_dev |
| dbt_cloud_pr_123_456 | None | dbt_cloud_pr_123_456 |
| analytics_prod | marketing | analytics_prod_marketing |
| alice_dev | marketing | alice_dev_marketing |
| dbt_cloud_pr_123_456 | marketing | dbt_cloud_pr_123_456_marketing |

## カスタムスキーマはどのように使用しますか？

モデルにカスタムスキーマを指定するには、`schema` 設定キーを使用します。他の設定と同様に、次のいずれかの操作を実行できます。

* モデル内の設定ブロックを使用して、特定のモデルにこの設定を適用する
* `dbt_project.yml` ファイルで指定して、モデルのサブディレクトリに適用する

<File name='orders.sql'>

```sql
{{ config(schema='marketing') }}

select ...
```

</File>

<File name='dbt_project.yml'>

```yaml
# models in `models/marketing/ will be built in the "*_marketing" schema
models:
  my_project:
    marketing:
      +schema: marketing
```

</File>

## カスタムスキーマについて

カスタムスキーマを初めて使用する場合、モデルは新しい `schema` 構成のみを使用するという誤解がよくあります。たとえば、`schema: marketing` 構成を持つモデルは `marketing` スキーマ内に構築されます。しかし、dbt はそれを `<target_schema>_marketing` のようなスキーマに配置します。

この相違には十分な理由があります。各 dbt ユーザーは、開発用に独自のターゲットスキーマを持っています ([環境の管理](#managing-environments) を参照)。dbt がターゲットスキーマを無視してモデルのカスタムスキーマのみを使用すると、すべての dbt ユーザーが同じスキーマ内にモデルを作成し、互いの作業を上書きしてしまいます。

ターゲットスキーマとカスタムスキーマを組み合わせることで、dbt はデータウェアハウス内に作成するオブジェクトが互いに衝突しないようにします。

スキーマ名を生成するために別のロジックを使用する場合は、dbt がスキーマ名を生成する方法を変更できます (以下を参照)。

### dbt はモデルのスキーマ名をどのように生成しますか？

dbt は、モデルを構築するスキーマ名を決定するために、`generate_schema_name` というデフォルトのマクロを使用します。

次のコードは、このデフォルトマクロのロジックを表しています:

```sql
{% macro generate_schema_name(custom_schema_name, node) -%}

    {%- set default_schema = target.schema -%}
    {%- if custom_schema_name is none -%}

        {{ default_schema }}

    {%- else -%}

        {{ default_schema }}_{{ custom_schema_name | trim }}

    {%- endif -%}

{%- endmacro %}
```
<br />

import WhitespaceControl from '/snippets/_whitespace-control.md';

<WhitespaceControl/>


## dbt によるスキーマ名生成方法の変更

dbt プロジェクトに `generate_schema_name` というカスタムマクロがある場合、dbt はデフォルトのマクロの代わりにこのマクロを使用します。これにより、ニーズに合わせて名前生成をカスタマイズできます。

このマクロをカスタマイズするには、[dbt によるモデルのスキーマ名の生成方法](#how-does-dbt-generate-a-models-schema-name) セクションのサンプルコードを `macros/generate_schema_name.sql` というファイルにコピーし、必要に応じて変更を加えます。

注意：dbt は、インストール済みパッケージに含まれるカスタム `generate_schema_name` マクロを無視します。

<Expandable alt_header="Warning: マクロ内の `default_schema` を置き換えないでください">

dbt によるスキーマ名の生成方法を変更する場合、```generate_schema_name``` マクロ内の ```{{ default_schema }}_{{ custom_schema_name | trim }}``` を ```{{ custom_schema_name | trim }}``` に置き換えるだけでは不十分です。

`{{ default_schema }}` を削除すると、開発者が独自のカスタムスキーマを作成する際に、互いのモデルをオーバーライドすることになります。これは、開発および継続的インテグレーション (CI) 中に問題を引き起こす可能性があります。

❌ 次のコード ブロックは、コードが次のようになってはならない例です:

```sql
{% macro generate_schema_name(custom_schema_name, node) -%}

    {%- set default_schema = target.schema -%}
    {%- if custom_schema_name is none -%}

        {{ default_schema }}

    {%- else -%}
    # The following is incorrect as it omits {{ default_schema }} before {{ custom_schema_name | trim }}. 
        {{ custom_schema_name | trim }} 

    {%- endif -%}

{%- endmacro %}

```

</Expandable>

### generate_schema_name 引数

| Argument | Description | Example |
| -------- | ----------- | ------- |
| custom_schema_name | 指定されたノードの `schema` の設定値、または値が指定されていない場合は `none` | `marketing` |
| node | 現在 dbt によって処理されている `node` | `{"name": "my_model", "resource_type": "model",...}` |

### generate_schema_name で利用可能な Jinja コンテキスト

スキーマ名を生成するためのカスタムロジックを記述する場合、そのロジックを定義する際にすべての変数とメソッドが利用できるわけではないことに注意してください。つまり、`generate_schema_name` マクロは、制限された Jinja コンテキストでコンパイルされます。

`generate_schema_name` マクロでは、以下のコンテキストメソッドが利用可能です:

| Jinja context | Type | Available |
| ------------- | ---- | --------- |
| [target](/reference/dbt-jinja-functions/target) | Variable | ✅ |
| [env_var](/reference/dbt-jinja-functions/env_var) | Variable | ✅ |
| [var](/reference/dbt-jinja-functions/var) | Variable | Limited, see below |
| [exceptions](/reference/dbt-jinja-functions/exceptions) | Macro | ✅ |
| [log](/reference/dbt-jinja-functions/log) | Macro | ✅ |
| プロジェクト内の他のマクロ | Macro | ✅ |
| パッケージ内の他のマクロ | Macro | ✅ |

### generate_schema_name で使用できる変数はどれですか？

グローバルスコープの変数と、コマンドラインで [--vars](/docs/build/project-variables) を使用して定義された変数は、`generate_schema_name` コンテキストでアクセスできます。

### パッケージ間で異なる動作を管理する

マクロ `dispatch` のドキュメントを参照してください: ["パッケージ間で異なるグローバルオーバーライドを管理する"](/reference/dbt-jinja-functions/dispatch)

## スキーマ名を生成するための組み込み代替パターン

一般的なカスタマイズ方法としては、カスタムスキーマが提供されている場合は本番環境でそれを使用し、カスタムスキーマが指定されていない場合はターゲットスキーマをフォールバックとしてのみ使用するというものがあります。開発環境やCIなどの他の環境では、カスタムスキーマの設定は無視され、代わりにターゲットスキーマがデフォルトとして使用されます。

Production Environment (`target.name == 'prod'`)

| Target schema | Custom schema | Resulting schema |
| ------------- | ------------- | ---------------- |
| analytics_prod | None | analytics_prod |
| analytics_prod | marketing | marketing |

Development/CI Environment (`target.name != 'prod'`)

| Target schema | Custom schema | Resulting schema |
| ------------- | ------------- | ---------------- |
| alice_dev | None | alice_dev |
| alice_dev | marketing | alice_dev |
| dbt_cloud_pr_123_456 | None | dbt_cloud_pr_123_456 |
| dbt_cloud_pr_123_456 | marketing | dbt_cloud_pr_123_456 |

通常のマクロと同様に、このアプローチにより、異なる環境のスキーマが衝突しないことが保証されます。

dbt には、このユースケース用のマクロ（「generate_schema_name_for_env」）が付属していますが、デフォルトでは無効になっています。有効にするには、次のコードを含むカスタムの「generate_schema_name」マクロをプロジェクトに追加します。

<File name='macros/get_custom_schema.sql'>

```sql
-- put this in macros/get_custom_schema.sql

{% macro generate_schema_name(custom_schema_name, node) -%}
    {{ generate_schema_name_for_env(custom_schema_name, node) }}
{%- endmacro %}
```

</File>

このマクロを使用する場合は、本番ジョブのターゲット名を `prod` に設定する必要があります。

## 環境の管理

[組み込み代替パターン](#スキーマ名生成のための組み込み代替パターン)セクションに示されている `generate_schema_name` マクロの例では、`target.name` コンテキスト変数を使用して、dbt がモデル用に生成するスキーマ名を変更しています。プロジェクトの `generate_schema_name` マクロで `target.name` コンテキスト変数を使用している場合は、各 dbt 環境が適切に構成されていることを確認する必要があります。任意の命名スキームを使用できますが、通常は次のスキームを推奨します。

* **dev** - ローカル開発環境。コンピュータ上の `profiles.yml` ファイルで設定されています。
* **ci** - GitHub、GitLab などのプルリクエストで実行される [継続的インテグレーション](/docs/cloud/git/connect-github) 環境。
* **prod** - dbt Cloud、Airflow、または[類似](/docs/deploy/deployments)などのdbtプロジェクトの本番環境デプロイメント。

スキーマ名が正しく生成されていない場合は、該当する環境でターゲット名を再確認してください。

詳細については、[dbt Core での環境管理](/docs/core/dbt-core-environments) ガイドをご覧ください。

## 関連ドキュメント

- [dbt モデルのデータベース、スキーマ、エイリアスのカスタマイズ](/guides/customize-schema-alias?step=1) で、dbt モデルのデータベース、スキーマ、エイリアスのカスタマイズ方法をご確認ください。
- [カスタム データベース](/docs/build/custom-databases) で、dbt モデル データベースのカスタマイズ方法をご確認ください。
- [カスタム エイリアス](/docs/build/custom-aliases) で、dbt モデルのエイリアス名のカスタマイズ方法をご確認ください。
