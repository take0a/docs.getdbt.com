---
title: "プロジェクト変数"
description: "Use dbt project variables to configure conditional or reusable logic across models and other resources." 
id: "project-variables"
pagination_next: "docs/build/environment-variables"
---

dbt は、モデルにコンパイル用のデータを提供するための [変数](/reference/dbt-jinja-functions/var) というメカニズムを提供しています。
変数は、[タイムゾーンを設定](https://github.com/dbt-labs/snowplow/blob/0.3.9/dbt_project.yml#L22)、[テーブル名のハードコーディングを回避](https://github.com/dbt-labs/quickbooks/blob/v0.1.0/dbt_project.yml#L23)、あるいはモデルにデータを提供してコンパイル方法を設定するために使用できます。

モデル、フック、またはマクロで変数を使用するには、`{{ var('...') }}` 関数を使用します。`var` 関数の詳細については、[こちら](/reference/dbt-jinja-functions/var) を参照してください。

変数は2つの方法で定義できます。

1. `dbt_project.yml` ファイル内
2. コマンドライン

### `dbt_project.yml` で変数を定義する


:::info

Jinja は `vars` 構成内ではサポートされていないため、すべての値は文字通り解釈されます。

:::


dbt プロジェクトで変数を定義するには、`dbt_project.yml` ファイルに `vars` 設定を追加します。
これらの `vars` は、グローバルにスコープすることも、プロジェクトにインポートされた特定のパッケージに限定することもできます。

<File name='dbt_project.yml'>

```yaml
name: my_dbt_project
version: 1.0.0

config-version: 2

vars:
  # The `start_date` variable will be accessible in all resources
  start_date: '2016-06-01'

  # The `platforms` variable is only accessible to resources in the my_dbt_project project
  my_dbt_project:
    platforms: ['web', 'mobile']

  # The `app_ids` variable is only accessible to resources in the snowplow package
  snowplow:
    app_ids: ['marketing', 'app', 'landing-page']

models:
    ...
```

</File>

### コマンドラインでの変数の定義

`dbt_project.yml` ファイルは、ほとんど変更されない変数を定義するのに最適です。
日付範囲など、頻繁に変更される変数もあります。
dbt の実行時に変数を定義（または上書き）するには、`--vars` コマンドラインオプションを使用します。
実際には、次のようになります。

```
$ dbt run --vars '{"key": "value"}'
```

`--vars` 引数は、コマンドラインで YAML 辞書を文字列として受け入れます。
YAML は、<Term id="json" /> のように厳密な引用符で囲む必要がないため便利です。

以下の 2 つの例はどちらも有効であり、同等です。

```
$ dbt run --vars '{"key": "value", "date": 20180101}'
$ dbt run --vars '{key: value, date: 20180101}'
```

設定する変数が 1 つだけの場合、括弧はオプションです。例:

```
$ dbt run --vars 'key: value'
```

YAML で辞書を定義する方法の詳細については、[こちら](https://github.com/Animosity/CraftIRC/wiki/Complete-idiot%27s-introduction-to-yaml) を参照してください。

### 変数の優先順位

`--vars` コマンドライン引数で定義された変数は、`dbt_project.yml` ファイルで定義された変数をオーバーライドします。これらの変数はグローバルスコープを持ち、ルートプロジェクトとインストールされたすべてのパッケージからアクセスできます。

変数宣言の優先順位は以下のとおりです（優先度の高いものから順に）。

1. コマンドラインで `--vars` を使用して定義された変数。
2. ルート `dbt_project.yml` ファイル内のパッケージスコープの変数宣言。
3. ルート `dbt_project.yml` ファイル内のグローバル変数宣言。
4. このノードがパッケージ内で定義されている場合：そのパッケージの `dbt_project.yml` ファイル内の変数宣言。
5. 変数のデフォルト引数（指定されている場合）

dbt が変数宣言の可能性のある場所をすべてチェックした後でも変数の定義を見つけられない場合、コンパイルエラーが発生します。

**注:** 変数のスコープは、その変数を最終的に使用するノードに基づきます。ルートプロジェクトで定義されたモデルが、インストール済みパッケージで定義されたマクロを呼び出す場合を想像してみてください。そのマクロは、変数の値を使用します。変数は、パッケージのスコープではなく、ルートプロジェクトのスコープに基づいて解決されます。

<Snippet path="discourse-help-feed-header" />
<DiscourseHelpFeed tags="variables"/>
