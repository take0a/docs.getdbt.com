---
resource_types: [models, seeds, snapshots, tests, analyses, metrics]
id: "group"
---

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
    { label: 'Tests', value: 'tests', },
    { label: 'Analyses', value: 'analyses', },
    { label: 'Metrics', value: 'metrics', },
    { label: 'Semantic models', value: 'semantic models', },
    { label: 'Saved queries', value: 'saved queries',} ,
  ]
}>
<TabItem value="models">
 
<File name='dbt_project.yml'>

```yml
models:

  [<resource-path>](resource-path):
    +group: GROUP_NAME

```


</File>

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: MODEL_NAME
    group: GROUP

```

</File>

<File name='models/<modelname>.sql'>

```sql

{{ config(
  group='GROUP_NAME'
) }}

select ...

```

</File>

</TabItem>

<TabItem value="seeds">

<File name='dbt_project.yml'>

```yml
models:
  [<resource-path>](resource-path):
    +group: GROUP_NAME
```

</File>

<File name='seeds/properties.yml'>

```yml
seeds:
  - name: [SEED_NAME]
    group: GROUP_NAME
```

</File>

</TabItem>

<TabItem value="snapshots">

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](resource-path):
    +group: GROUP_NAME
```

</File>

<VersionBlock firstVersion="1.9">
<File name='snapshots/properties.yml'>

```yaml
version: 2

snapshots:
  - name: snapshot_name
    [config](/reference/resource-properties/config):
      group: GROUP_NAME
```

</File>
</VersionBlock>

<File name='snapshots/<filename>.sql'>

```sql
{% snapshot [snapshot_name](snapshot_name) %}

{{ config(
  group='GROUP_NAME'
) }}

select ...

{% endsnapshot %}
```

</File>

</TabItem>

<TabItem value="tests">

<File name='dbt_project.yml'>

```yml
tests:
  [<resource-path>](resource-path):
    +group: GROUP_NAME
```

</File>

<File name='tests/properties.yml'>

```yml
version: 2

<resource_type>:
  - name: <resource_name>
    tests:
      - <test_name>:
          config:
            group: GROUP_NAME
```

</File>

<File name='tests/<filename>.sql'>

```sql
{% test <testname>() %}

{{ config(
  group='GROUP_NAME'
) }}

select ...

{% endtest %}
```

</File>

<File name='tests/<filename>.sql'>


```sql
{{ config(
  group='GROUP_NAME'
) }}
```

</File>

</TabItem>

<TabItem value="analyses">

<File name='analyses/<filename>.yml'>

```yml
version: 2

analyses:
  - name: ANALYSIS_NAME
    group: GROUP_NAME
```

</File>

</TabItem>


<TabItem value="metrics">

<File name='dbt_project.yml'>

```yaml
metrics:
  [<resource-path>](resource-path):
    [+](plus-prefix)group: GROUP_NAME
```

</File>

<File name='models/metrics.yml'>

```yaml
version: 2

metrics:
  - name: [METRIC_NAME]
    config:
      group: GROUP_NAME

```

</File>

</TabItem>


<TabItem value="semantic models">

<File name='dbt_project.yml'>

```yaml
semantic-models:
  [<resource-path>](resource-path):
    [+](plus-prefix)group: GROUP_NAME
```

</File>

<File name='models/semantic_models.yml'>

```yaml
semantic_models:
  - name: SEMANTIC_MODEL_NAME
    config:
      group: GROUP_NAME
```

</File>

</TabItem>

<TabItem value="saved queries">

<File name='dbt_project.yml'>

```yaml
saved-queries:
  [<resource-path>](resource-path):
    [+](plus-prefix)group: GROUP_NAME
```

</File>

<File name='models/semantic_models.yml'>

```yaml
saved_queries:
  - name: SAVED_QUERY_NAME
    config:
      group: GROUP_NAME
```

</File>

</TabItem>

</Tabs>

## 定義

リソースにグループを割り当てるためのオプションの設定です。リソースをグループ化すると、dbt は同じグループ内のプライベートモデルへの参照を許可します。

グループ内のリソース間の参照アクセスの詳細については、[モデルアクセス](/docs/collaborate/govern/model-access#groups) をご覧ください。

## 例

### 「マーケティング」グループモデルがプライベートな「財務」グループモデルを参照するのを防ぎます。
これは、急速に変化するモデル、実験的なモデル、あるいはグループやチーム内部のモデルを他のグループが利用できないようにする場合に役立ちます。

<File name='models/schema.yml'>

```yml
models:
  - name: finance_model
    access: private
    group: finance
  - name: marketing_model
    group: marketing
```
</File>

<File name='models/marketing_model.sql'>

```sql
select * from {{ ref('finance_model') }}
```
</File>

```shell
$ dbt run -s marketing_model
...
dbt.exceptions.DbtReferenceError: Parsing Error
  Node model.jaffle_shop.marketing_model attempted to reference node model.jaffle_shop.finance_model, 
  which is not allowed because the referenced node is private to the finance group.
```

## 関連ドキュメント

* [モデルアクセス](/docs/collaborate/govern/model-access#groups)
* [グループの定義](/docs/build/groups)
