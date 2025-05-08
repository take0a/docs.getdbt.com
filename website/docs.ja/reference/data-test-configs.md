---
title: Data test 構成
description: "dbt で data test 構成を使用する方法については、このガイドをお読みください。"
meta:
  resource_type: Data tests
---
import ConfigResource from '/snippets.ja/_config-description-resource.md';
import ConfigGeneral from '/snippets.ja/_config-description-general.md';


## 関連ドキュメント

* [データテスト](/docs/build/data-tests)

データテストは、いくつかの方法で設定できます。
1. `.yml` 定義内のプロパティ（汎用テストのみ。完全な構文については、[テストプロパティ](/reference/resource-properties/data-tests) を参照してください）
2. テストの SQL 定義内の `config()` ブロック
3. `dbt_project.yml` 内

データテストの設定は、上記の詳細度に基づいて階層的に適用されます。単一のテストの場合、SQL 定義内の `config()` ブロックがプロジェクトファイル内の設定よりも優先されます。汎用テストの特定のインスタンスの場合、テストの `.yml` プロパティは、汎用 SQL 定義の `config()` で設定された値よりも優先され、さらに `dbt_project.yml` で設定された値よりも優先されます。

## 利用可能な構成

各構成オプションのリンクをクリックすると、その機能の詳細をご覧いただけます。

### Data test-specific configurations

<ConfigResource meta={frontMatter.meta} />

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Config block', value: 'config', },
    { label: 'Property file', value: 'property-yaml', },
  ]
}>
<TabItem value="project-yaml">

<File name='dbt_project.yml'>

```yaml
tests:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[fail_calc](/reference/resource-configs/fail_calc): <string>
    [+](/reference/resource-configs/plus-prefix)[limit](/reference/resource-configs/limit): <integer>
    [+](/reference/resource-configs/plus-prefix)[severity](/reference/resource-configs/severity): error | warn
    [+](/reference/resource-configs/plus-prefix)[error_if](/reference/resource-configs/severity): <string>
    [+](/reference/resource-configs/plus-prefix)[warn_if](/reference/resource-configs/severity): <string>
    [+](/reference/resource-configs/plus-prefix)[store_failures](/reference/resource-configs/store_failures): true | false
    [+](/reference/resource-configs/plus-prefix)[where](/reference/resource-configs/where): <string>

```

</File>

</TabItem>


<TabItem value="config">

```jinja

{{ config(
    [fail_calc](/reference/resource-configs/fail_calc) = "<string>",
    [limit](/reference/resource-configs/limit) = <integer>,
    [severity](/reference/resource-configs/severity) = "error | warn",
    [error_if](/reference/resource-configs/severity) = "<string>",
    [warn_if](/reference/resource-configs/severity) = "<string>",
    [store_failures](/reference/resource-configs/store_failures) = true | false,
    [where](/reference/resource-configs/where) = "<string>"
) }}

```


</TabItem>

<TabItem value="property-yaml">

```yaml
version: 2

<resource_type>:
  - name: <resource_name>
    tests:
      - <test_name>: # # Actual name of the test. For example, dbt_utils.equality
          name: # Human friendly name for the test. For example, equality_fct_test_coverage
          [description](/reference/resource-properties/description): "markdown formatting"
          <argument_name>: <argument_value>
          [config](/reference/resource-properties/config):
            [fail_calc](/reference/resource-configs/fail_calc): <string>
            [limit](/reference/resource-configs/limit): <integer>
            [severity](/reference/resource-configs/severity): error | warn
            [error_if](/reference/resource-configs/severity): <string>
            [warn_if](/reference/resource-configs/severity): <string>
            [store_failures](/reference/resource-configs/store_failures): true | false
            [where](/reference/resource-configs/where): <string>

    [columns](/reference/resource-properties/columns):
      - name: <column_name>
        tests:
          - <test_name>:
              name: 
              [description](/reference/resource-properties/description): "markdown formatting"
              <argument_name>: <argument_value>
              [config](/reference/resource-properties/config):
                [fail_calc](/reference/resource-configs/fail_calc): <string>
                [limit](/reference/resource-configs/limit): <integer>
                [severity](/reference/resource-configs/severity): error | warn
                [error_if](/reference/resource-configs/severity): <string>
                [warn_if](/reference/resource-configs/severity): <string>
                [store_failures](/reference/resource-configs/store_failures): true | false
                [where](/reference/resource-configs/where): <string>
```

この設定メカニズムは、汎用テストの特定のインスタンスに対してのみサポートされます。特定のテストを設定するには、SQL定義で `config()` マクロを使用する必要があります。


</TabItem>

</Tabs>


### 一般的な構成

<ConfigGeneral />

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Config block', value: 'config', },
    { label: 'Property file', value: 'property-yaml', },
  ]
}>
<TabItem value="project-yaml">


<File name='dbt_project.yml'>

```yaml
tests:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta): {dictionary}
    # relevant for [store_failures](/reference/resource-configs/store_failures) only
    [+](/reference/resource-configs/plus-prefix)[database](/reference/resource-configs/database): <string>
    [+](/reference/resource-configs/plus-prefix)[schema](/reference/resource-properties/schema): <string>
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
```
</File>

</TabItem>

<TabItem value="config">


```jinja

{{ config(
    [enabled](/reference/resource-configs/enabled)=true | false,
    [tags](/reference/resource-configs/tags)="<string>" | ["<string>"]
    [meta](/reference/resource-configs/meta)={dictionary},
    [database](/reference/resource-configs/database)="<string>",
    [schema](/reference/resource-properties/schema)="<string>",
    [alias](/reference/resource-configs/alias)="<string>",
) }}

```

</TabItem>

<TabItem value="property-yaml">

```yaml
version: 2

<resource_type>:
  - name: <resource_name>
    tests:
      - <test_name>: # Actual name of the test. For example, dbt_utils.equality
          name: # Human friendly name for the test. For example, equality_fct_test_coverage
          [description](/reference/resource-properties/description): "markdown formatting"
          <argument_name>: <argument_value>
          [config](/reference/resource-properties/config):
            [enabled](/reference/resource-configs/enabled): true | false
            [tags](/reference/resource-configs/tags): <string> | [<string>]
            [meta](/reference/resource-configs/meta): {dictionary}
            # relevant for [store_failures](/reference/resource-configs/store_failures) only
            [database](/reference/resource-configs/database): <string>
            [schema](/reference/resource-properties/schema): <string>
            [alias](/reference/resource-configs/alias): <string>

    [columns](/reference/resource-properties/columns):
      - name: <column_name>
        tests:
          - <test_name>:
              name: 
              [description](/reference/resource-properties/description): "markdown formatting"
              <argument_name>: <argument_value>
              [config](/reference/resource-properties/config):
                [enabled](/reference/resource-configs/enabled): true | false
                [tags](/reference/resource-configs/tags): <string> | [<string>]
                [meta](/reference/resource-configs/meta): {dictionary}
                # relevant for [store_failures](/reference/resource-configs/store_failures) only
                [database](/reference/resource-configs/database): <string>
                [schema](/reference/resource-properties/schema): <string>
                [alias](/reference/resource-configs/alias): <string>
```

この設定メカニズムは、汎用データテストの特定のインスタンスに対してのみサポートされます。特定のテストを設定するには、SQL定義で `config()` マクロを使用する必要があります。


</TabItem>


</Tabs>

### 例

#### 1つのテストにタグを追加する

汎用データテストの特定のインスタンスの場合:

<File name='models/<filename>.yml'>

```yml
models:
  - name: my_model
    columns:
      - name: id
        tests:
          - unique:
              tags: ['my_tag']
```

</File>

特異データテストの場合:

<File name='tests/<filename>.sql'>

```sql
{{ config(tags = ['my_tag']) }}

select ...
```

</File>

#### 汎用データテストのすべてのインスタンスのデフォルトの重大度を設定します

<File name='macros/<filename>.sql'>

```sql
{% test my_test() %}

    {{ config(severity = 'warn') }}

    select ...

{% endtest %}
```

</File>

#### パッケージからすべてのデータテストを無効にする

<File name='dbt_project.yml'>

```yml
tests:
  package_name:
    +enabled: false
```

</File>

#### 汎用データテストのカスタム設定の指定

dbt v1.9 以降では、任意のカスタム設定キーを使用してデータテストのカスタム設定を指定できます。例えば、次の例は、dbt が `accepted_values` データテストを実行する際に使用する `snowflake_warehouse` カスタム設定を指定します:

```yml

models:
  - name: my_model
    columns:
      - name: color
        tests:
          - accepted_values:
              values: ['blue', 'red']
              config:
                severity: warn
                snowflake_warehouse: my_warehouse

```

構成が指定されると、データ テストは、デフォルトの接続とは異なる Snowflake 仮想ウェアハウスで実行され、異なるウェアハウス サイズやよりきめ細かなコスト割り当てと可視性によって、より優れた価格パフォーマンスが実現されます。

#### 汎用テストと特異テストに説明を追加する

dbt v1.9（dbt Cloud [リリーストラック](/docs/dbt-versions/cloud-release-tracks)でも利用可能）以降では、汎用テストと特異テストの両方に[説明](/reference/resource-properties/data-tests#description)を追加できます。

汎用テストの場合は、既存のYAMLに合わせて説明を追加します:

<File name='models/staging/<filename>.yml'>

```yml

models:
  - name: my_model
    columns:
      - name: delivery_status
        tests:
          - accepted_values:
              values: ['delivered', 'pending', 'failed']
              description: "This test checks whether there are unexpected delivery statuses. If it fails, check with logistics team"

```
</File>

汎用データテストのコアロジックを提供するJinjaマクロに説明を追加することもできます。詳細については、[汎用データテストのロジックに説明を追加する](/best-practices/writing-custom-generic-tests#add-description-to-generic-data-test-logic)を参照してください。

単一のテストの場合は、テストのディレクトリに定義します:

<File name='tests/my_custom_test.yml'>

```yml

data_tests: 
  - name: my_custom_test
    description: "This test checks whether the rolling average of returns is inside of expected bounds. If it isn't, flag to customer success team"

```
</File>

詳細については、「データ テストに説明を追加する」 (/reference/resource-properties/description#add-a-description-to-a-data-test) を参照してください。

