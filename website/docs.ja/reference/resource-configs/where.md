---
resource_types: [tests]
datatype: string
---

### 定義

テスト対象のリソース（モデル、ソース、シード、またはスナップショット）をフィルタリングします。

`where` 条件は、リソース参照を <Term id="subquery" /> に置き換えることでテストクエリにテンプレート化されます。例えば、`not_null` テストは次のようになります。

```sql
select *
from my_model
where my_column is null
```

`where` 構成が `where date_column = current_date` に設定されている場合、テスト クエリは次のように更新されます。

```sql
select *
from (select * from my_model where date_column = current_date) dbt_subquery
where my_column is null
```

### 例

<Tabs
  defaultValue="specific"
  values={[
    { label: 'Specific test', value: 'specific', },
    { label: 'One-off test', value: 'one_off', },
    { label: 'Generic test block', value: 'generic', },
    { label: 'Project level', value: 'project', },
  ]
}>

<TabItem value="specific">

汎用 (スキーマ) テストの特定のインスタンスを構成します:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: large_table
    columns:
      - name: my_column
        tests:
          - accepted_values:
              values: ["a", "b", "c"]
              config:
                where: "date_column = current_date"
      - name: other_column
        tests:
          - not_null:
              where: "date_column < current_date"
```

</File>

</TabItem>

<TabItem value="one_off">

この設定は、1 回限りのテストでは無視されます。

</TabItem>

<TabItem value="generic">

テスト ブロック (定義) 内に構成を設定して、汎用 (スキーマ) テストのすべてのインスタンスのデフォルトを設定します:

<File name='macros/<filename>.sql'>

```sql
{% test <testname>(model, column_name) %}

{{ config(where = "date_column = current_date") }}

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
  +where: "date_column = current_date"
  
  <package_name>:
    +where: >
        date_column = current_date
        and another_column is not null
```

</File>

</TabItem>

</Tabs>

### カスタムロジック

`where` 構成のレンダリングコンテキストは、`.yml` ファイルで定義されたすべての構成と同じです。`{{ var() }}` と `{{ env_var() }}` は使用できますが、この構成を設定するためのカスタムマクロは使用できません。カスタムマクロを使用して特定のテストの `where` フィルターをテンプレート化したい場合は、回避策があります。

dbt は `get_where_subquery` マクロを定義します。

dbt は、汎用テスト定義内の `{{ model }}` を `{{ get_where_subquery(relation) }}` に置き換えます。ここで、`relation` はテスト対象リソースの `ref()` または `source()` です。このマクロのデフォルト実装では、以下を返します。
- `where` 設定が定義されていない場合 (`ref()` または `source()`)、`{{ relationship }}` を返します。
- `where` 設定が定義されている場合、`(select * from {{ relationship }} where {{ where }}) dbt_subquery` を返します。

この動作は、以下の方法でオーバーライドできます。
- ルートプロジェクトでカスタム `get_where_subquery` を定義する。
- パッケージまたはアダプタプラグインでカスタム `<adapter>__get_where_subquery` [ディスパッチ候補](/reference/dbt-jinja-functions/dispatch) を定義する。

このマクロ定義内では、設定からの静的入力に基づいて、任意のカスタムマクロを参照できます。簡単に言えば、これにより、多くの異なる `.yml` ファイルで繰り返し記述する必要のあるコードを DRY 化できます。 `get_where_subquery` マクロは実行時に解決されるため、カスタム マクロには [イントロスペクティブ データベース クエリの結果の取得](https://docs.getdbt.com/reference/dbt-jinja-functions/run_query) も含めることができます。

#### 例

dbtのクロスプラットフォーム [`dateadd()`](/reference/dbt-jinja-functions/cross-database-macros#dateadd) ユーティリティマクロを使用して、テストを過去N日間のデータにフィルタリングします。プレースホルダ文字列で日数を設定できます。

<File name='models/config.yml'>

```yml
version: 2
models:
  - name: my_model
    columns:
      - name: id
        tests:
          - unique:
              config:
                where: "date_column > __3_days_ago__"  # placeholder string for static config
```

</File>

<File name='macros/custom_get_where_subquery.sql'>

```sql
{% macro get_where_subquery(relation) -%}
    {% set where = config.get('where') %}
    {% if where %}
        {% if "_days_ago__" in where %}
            {# replace placeholder string with result of custom macro #}
            {% set where = replace_days_ago(where) %}
        {% endif %}
        {%- set filtered -%}
            (select * from {{ relation }} where {{ where }}) dbt_subquery
        {%- endset -%}
        {% do return(filtered) %}
    {%- else -%}
        {% do return(relation) %}
    {%- endif -%}
{%- endmacro %}

{% macro replace_days_ago(where_string) %}
    {# Use regex to search the pattern for the number days #}
    {# Default to 3 days when no number found #}
    {% set re = modules.re %}
    {% set days = 3 %}
    {% set pattern = '__(\d+)_days_ago__' %}
    {% set match = re.search(pattern, where_string) %}
    {% if match %}
        {% set days = match.group(1) | int %}        
    {% endif %}
    {% set n_days_ago = dbt.dateadd('day', -days, current_timestamp()) %}
    {% set result = re.sub(pattern, n_days_ago, where_string) %}
    {{ return(result) }}
{% endmacro %}
```

</File>
