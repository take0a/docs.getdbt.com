---
resource_types: [tests]
datatype: string
---

テストクエリは、テストで宣言された期待値やアサーションに一致しない、失敗したレコードのセットを返すように記述されます。これには重複レコードや null 値などが含まれます。

多くの場合、これはテストクエリによって返される行数です。`fail_calc` のデフォルト値は `count(*)` です。ただし、集計計算やテストクエリから選択される列名など、カスタム計算を指定することもできます。

ほとんどのテストでは `fail_calc` 設定を使用せず、失敗した行数を返すことを優先します。`fail_calc` 設定を使用するテストでは、`fail_calc` 設定を設定する最も一般的な場所は、汎用テストブロック内、クエリ定義と一緒の場所です。同様に、`fail_calc` は他の設定と同様に設定できます。

たとえば、失敗の計算として `count(*)` ではなく `sum(n_records)` を返すように `unique` テストを設定できます。つまり、重複している個別の列値の数ではなく、重複した列値を含むモデル内の行の数です。

:::tip Tip
行が返されない可能性のあるテストでは、`fail_calc` に `sum()` などの関数を使用することに注意してください。

行が返されない場合、テストは成功も失敗も判定されず、次のエラーが返されます:

```
None is not of type 'integer'

Failed validating 'type' in schema['properties']['failures']:
    {'type': 'integer'}

On instance['failures']:
    None
```

この問題を回避するには、case ステートメントを使用して、行が存在しない場合に `0` が返されるようにします:

```yaml
fail_calc: "case when count(*) > 0 then sum(n_records) else 0 end"
```

:::

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
  - name: my_model
    columns:
      - name: my_columns
        tests:
          - unique:
              config:
                fail_calc: "case when count(*) > 0 then sum(n_records) else 0 end"
```

</File>

</TabItem>

<TabItem value="one_off">

1 回限りの (データ) テストを構成します:

<File name='tests/<filename>.sql'>

```sql
{{ config(fail_calc = "sum(total_revenue) - sum(revenue_accounted_for)") }}

select ...
```

</File>

</TabItem>

<TabItem value="generic">

テスト ブロック (定義) 内に構成を設定して、汎用 (スキーマ) テストのすべてのインスタンスのデフォルトを設定します:

<File name='macros/<filename>.sql'>

```sql
{% test <testname>(model, column_name) %}

{{ config(fail_calc = "missing_in_a + missing_in_b") }}

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
  +fail_calc: count(*)  # all tests
  
  <package_name>:
    +fail_calc: count(distinct id) # tests in <package_name>
```

</File>

</TabItem>

</Tabs>
