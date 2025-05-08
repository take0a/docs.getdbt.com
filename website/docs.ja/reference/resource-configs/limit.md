---
resource_types: [tests]
datatype: integer
---

テストクエリによって返される失敗の数を制限します。大規模なデータセットを扱い、[失敗をデータベースに保存](/reference/resource-configs/store_failures)する場合は、この設定を使用することをお勧めします:

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
      - name: very_unreliable_column
        tests:
          - accepted_values:
              values: ["a", "b", "c"]
              config:
                limit: 1000  # will only include the first 1000 failures
```

</File>

</TabItem>

<TabItem value="one_off">

1 回限りの (データ) テストを構成します:

<File name='tests/<filename>.sql'>

```sql
{{ config(limit = 1000) }}

select ...
```

</File>

</TabItem>

<TabItem value="generic">

テスト ブロック (定義) 内に構成を設定して、汎用 (スキーマ) テストのすべてのインスタンスのデフォルトを設定します:

<File name='macros/<filename>.sql'>

```sql
{% test <testname>(model, column_name) %}

{{ config(limit = 500) }}

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
  +limit: 1000  # all tests
  
  <package_name>:
    +limit: 50 # tests in <package_name>
```

</File>

</TabItem>

</Tabs>
