---
title: severity, error_if, and warn_if
id: "severity"
description: "エラーしきい値を使用して、テスト結果の重大度を構成し、失敗したテストの数に基づいてエラーまたは警告を生成するタイミングを設定できます。"
resource_types: [tests]
datatype: string
keywords: [severity, error_if, warn_if]
---

テストは失敗の数を返します。これは通常、テストクエリによって返された行数ですが、[カスタム計算](/reference/resource-configs/fail_calc) が使用される場合もあります。一般的に、失敗の数が0以外の場合、テストはエラーを返します。テストクエリは、重複レコードやnull値など、不要な行をすべて返すように設計されているため、これは理にかなっています。

テストを設定して、エラーではなく警告を返したり、返された失敗の数に応じてテストのステータスを条件付きにしたりすることも可能です。重複レコードが1件であれば警告としてカウントできますが、重複レコードが10件であればエラーとしてカウントする必要があります。

関連する設定は次のとおりです。
- `severity`: `error` または `warn` (デフォルト: `error`)
- `error_if`: 条件式 (デフォルト: `!=0`)
- `warn_if`: 条件式 (デフォルト: `!=0`)

条件式は、SQL 構文でサポートされている任意の比較ロジックで、失敗回数は整数で指定します。`> 5`、`= 0`、`between 5 and 10` などです。

実際の動作は次のとおりです。
- `severity: error` の場合、dbt はまず `error_if` 条件をチェックします。エラー条件が満たされた場合、テストはエラーを返します。満たされなかった場合、dbt は次に `warn_if` 条件 (デフォルト: `!=0`) をチェックします。指定されていない場合、または警告条件が満たされた場合、テストは警告を出力します。条件が満たされていない場合、テストは成功します。
- `severity: warn` の場合、dbt は `error_if` 条件を完全にスキップし、直接 `warn_if` 条件に進みます。warn 条件が満たされた場合、テストは警告を出力し、満たされていない場合、テストは成功します。

[`--warn-error`](/reference/global-configs/warnings) フラグが指定されている場合、テストの警告ステータスはエラーを返すことに注意してください。dbt に警告をエラーとして扱うように指示しない限り、重大度が `warn` のテストはエラーを返すことはありません。

<Tabs
  defaultValue="generic"
  values={[
    { label: 'Out-of-the-box generic tests', value: 'generic', },
    { label: 'Singular tests', value: 'singular', },
    { label: 'Custom generic tests', value: 'custom-generic', },
    { label: 'Project level', value: 'project', },
  ]
}>

<TabItem value="generic">

すぐに使用できる汎用テストの特定のインスタンスを構成します:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: large_table
    columns:
      - name: slightly_unreliable_column
        tests:
          - unique:
              config:
                severity: error
                error_if: ">1000"
                warn_if: ">10"
```

</File>

</TabItem>

<TabItem value="singular">

単一テストを構成します:

<File name='tests/<filename>.sql'>

```sql
{{ config(error_if = '>50') }}

select ...
```

</File>

</TabItem>

<TabItem value="custom-generic">

テスト ブロック (定義) 内に構成を設定して、カスタム汎用テストのすべてのインスタンスのデフォルトを設定します:

<File name='macros/<filename>.sql'>

```sql
{% test <testname>(model, column_name) %}

{{ config(severity = 'warn') }}

select ...

{% endtest %}
```

</File>

</TabItem>

<TabItem value="project">

パッケージまたはプロジェクト内のすべてのテストのデフォルトを設定します:

<File name='dbt_project.yml'>

```yaml
tests:
  +severity: warn  # all tests

  <package_name>:
    +warn_if: >10 # tests in <package_name>
```

</File>

</TabItem>

</Tabs>
