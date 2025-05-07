---
resource_types: all
datatype: "{<dictionary>}"
default_value: {}
hide_table_of_contents: true
---

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Sources', value: 'sources', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
    { label: 'Tests', value: 'tests', },
    { label: 'Analyses', value: 'analyses', },
    { label: 'Macros', value: 'macros', },
    { label: 'Exposures', value: 'exposures', },
    { label: 'Semantic models', value: 'semantic models', },
    { label: 'Metrics', value: 'metrics', },
    { label: 'Saved queries', value: 'saved queries', },
  ]
}>
<TabItem value="models">

<File name='dbt_project.yml'>

```yml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}

```

</File>

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: model_name
    config:
      meta: {<dictionary>}

    columns:
      - name: column_name
        meta: {<dictionary>}

```

</File>

`meta` 設定は、以下の場所でも定義できます。
- `dbt_project.yml` の `models` 設定ブロック内
- モデルの SQL ファイル内の `config()` Jinja マクロ内

詳細については、[設定とプロパティ](/reference/configs-and-properties) を参照してください。

</TabItem>

<TabItem value="sources">

<File name='dbt_project.yml'>

```yml
sources:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='models/schema.yml'>

```yml
version: 2

[sources](/reference/source-properties):
  - name: model_name
    config:
      meta: {<dictionary>}

    tables:
      - name: table_name
        config:
          meta: {<dictionary>}

        columns:
          - name: column_name
            meta: {<dictionary>}

```

</File>

</TabItem>

<TabItem value="seeds">

<File name='dbt_project.yml'>

```yml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='seeds/schema.yml'>

```yml
version: 2

seeds:
  - name: seed_name
    config:
      meta: {<dictionary>}

    columns:
      - name: column_name
        meta: {<dictionary>}

```

</File>

`meta` 設定は、`dbt_project.yml` の `seeds` 設定ブロック内でも定義できます。詳細は [設定とプロパティ](/reference/configs-and-properties) を参照してください。

</TabItem>

<TabItem value="snapshots">

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='snapshots/schema.yml'>

```yml
version: 2

snapshots:
  - name: snapshot_name
    config:
      [meta](/reference/snapshot-properties): {<dictionary>}

    columns:
      - name: column_name
        meta: {<dictionary>}

```

</File>

`meta` 設定は、以下の場所でも定義できます。
- `dbt_project.yml` の `snapshots` 設定ブロック内
- スナップショットの SQL ブロック内の `config()` Jinja マクロ内

詳細については、[設定とプロパティ](/reference/configs-and-properties) を参照してください。

</TabItem>

<TabItem value="tests">

[汎用テスト](/docs/build/data-tests#generic-data-tests)にはYAMLの`meta`設定を追加できません。ただし、[特異なテスト](/docs/build/data-tests#singular-data-tests)には、テストファイルの先頭で`config()`を使用することで、`meta`プロパティを追加できます。

</TabItem>

<TabItem value="analyses">

`meta` 設定は現在、analyses ではサポートされていません。

</TabItem>

<TabItem value="macros">

<File name='dbt_project.yml'>

```yml
macros:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='macros/schema.yml'>

```yml
version: 2

[macros](/reference/macro-properties):
  - name: macro_name
    meta: {<dictionary>}

    arguments:
      - name: argument_name

```

</File>

</TabItem>

<TabItem value="exposures">

<File name='dbt_project.yml'>

```yml
exposures:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='models/exposures.yml'>

```yml
version: 2

exposures:
  - name: exposure_name
    meta: {<dictionary>}

```

</File>

</TabItem>

<TabItem value="semantic models">

[セマンティック モデル](/docs/build/semantic-models) YAML ファイル内、または `dbt_project.yml` ファイルの `semantic-models` 構成ブロック内で `meta` を構成します。

<VersionBlock lastVersion="1.9">

<File name='models/semantic_models.yml'>

```yml
semantic_models:
  - name: semantic_model_name
    config:
      meta: {<dictionary>}

```

</File>

<File name='dbt_project.yml'>

```yml
semantic-models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

</VersionBlock>

<VersionBlock firstVersion="1.9">


<File name='dbt_project.yml'>

```yml
semantic-models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='models/semantic_models.yml'>

```yml
semantic_models:
  - name: semantic_model_name
    config:
      meta: {<dictionary>}

```

</File>

[ディメンション](/docs/build/dimensions)、[エンティティ](/docs/build/entities)、[メジャー](/docs/build/measures) にも独自の `meta` 構成を設定できます。

<File name='models/semantic_models.yml'>

```yml
semantic_models:
  - name: semantic_model_name
    config:
      meta: {<dictionary>}

    dimensions:
      - name: dimension_name
        config:
          meta: {<dictionary>}

    entities:
      - name: entity_name
        config:
          meta: {<dictionary>}

    measures:
      - name: measure_name
        config:
          meta: {<dictionary>}

```

</File>

</VersionBlock>

`meta` 設定は、`dbt_project.yml` の `semantic-models` 設定ブロックでも定義できます。詳細は [設定とプロパティ](/reference/configs-and-properties) を参照してください。

</TabItem>

<TabItem value="metrics">

<VersionBlock lastVersion="1.7">

<File name='dbt_project.yml'>

```yml
metrics:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='models/metrics.yml'>

```yml
metrics:
  - name: number_of_people
    label: "Number of people"
    description: Total count of people
    type: simple
    type_params:
      measure: people
    meta:
      my_meta_direct: 'direct'
```

</File>
</VersionBlock>

<VersionBlock firstVersion="1.8"> 

<File name='dbt_project.yml'>

```yml
metrics:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='models/metrics.yml'>

```yml
metrics:
  - name: number_of_people
    label: "Number of people"
    description: Total count of people
    type: simple
    type_params:
      measure: people
    config:
      meta:
        my_meta_config: 'config_value'
```

</File>
</VersionBlock>

</TabItem>

<TabItem value="saved queries">

<File name='dbt_project.yml'>

```yml
saved-queries:
  [<resource-path>](/reference/resource-configs/resource-path):
    +meta: {<dictionary>}
```
</File>

<File name='models/semantic_models.yml'>

```yml
saved_queries:
  - name: saved_query_name
    config:
      meta: {<dictionary>}
```

</File>
</TabItem>
</Tabs>

## 定義

`meta` フィールドはリソースのメタデータを設定するために使用でき、任意のキーと値のペアを受け入れます。このメタデータは、dbt によって生成される `manifest.json` ファイルにコンパイルされ、自動生成されるドキュメントで確認できます。

設定するリソースによっては、`meta` が `config` プロパティ内、または最上位キーとして使用できる場合があります。(後方互換性のため、`meta` は多くの場合 (常にではありませんが) 最上位キーとしてサポートされますが、設定の継承機能は利用できません。)


## 例

### モデルオーナーを指定します。

さらに、「model_maturity:」キーを使用してモデルの成熟度を示します。

<File name='models/schema.yml'>

```yml
version: 2

models:
  - name: users
    meta:
      owner: "@alice"
      model_maturity: in dev

```

</File>


### ソース列にPIIが含まれるように指定する

<File name='models/schema.yml'>

```yml
version: 2

[sources](/reference/source-properties):
  - name: salesforce

    tables:
      - name: account
        meta:
          contains_pii: true
        columns:
          - name: email
            meta:
              contains_pii: true

```

</File>

### すべてのシードに対して1つのメタ属性を設定する

<File name='dbt_project.yml'>

```yml
seeds:
  +meta:
    favorite_color: red
```

</File>

### 1つのモデルの1つのメタ属性をオーバーライドする

<File name='models/my_model.sql'>

```sql
{{ config(meta = {
    'single_key': 'override'
}) }}

select 1 as id
```

</File><br />

### dbt_project.yml に owner と favorite_color を設定プロパティとして割り当てます。

<File name='dbt_project.yml'>

```yml
models:
  jaffle_shop:
    +meta:
      owner: "@alice"
      favorite_color: red
```

</File>

### セマンティックモデルにメタ値を割り当てる

次の例は、`semantic_model.yml` ファイルと `dbt_project.yml` ファイルで [セマンティックモデル](/docs/build/semantic-models) に `meta` 値を割り当てる方法を示しています。

<Tabs>
<TabItem value="semantic_model" label="Semantic model">

```yaml
semantic_models:
  - name: transaction 
    model: ref('fact_transactions')
    description: "Transaction fact table at the transaction level. This table contains one row per transaction and includes the transaction timestamp."
    defaults:
      agg_time_dimension: transaction_date
    config:
      meta:
        data_owner: "Finance team"
        used_in_reporting: true
```

</TabItem>

<TabItem value="project.yml" label="dbt_project.yml">

```yaml
semantic-models:
  jaffle_shop:
    +meta:
      used_in_reporting: true
```
</TabItem>
</Tabs>

### ディメンション、メジャー、エンティティにメタを割り当てる

<VersionBlock lastVersion="1.8">

Available in dbt version 1.9 and later.

</VersionBlock>

<VersionBlock firstVersion="1.9">

<Tabs>
<TabItem value="semantic_model" label="Semantic model">

次の例は、セマンティック モデル内の [ディメンション](/docs/build/dimensions)、[エンティティ](/docs/build/entities)、および [メジャー](/docs/build/measures) に `meta` 値を割り当てる方法を示しています:

<File name='semantic_model.yml'>

```yml
semantic_models:
  - name: semantic_model
    ...
    dimensions:
      - name: order_date
        type: time
        config:
          meta:
            data_owner: "Finance team"
            used_in_reporting: true
    entities:
      - name: customer_id
        type: primary
        config:
          meta:
            description: "Unique identifier for customers"
            data_owner: "Sales team"
            used_in_reporting: false
    measures:
      - name: count_of_users
        expr: user_id
        config:
          meta:
            used_in_reporting: true
```

</File>
</TabItem>

<TabItem value="project.yml" label="dbt_project.yml">

この2番目の例は、`dbt_project.yml`ファイル内のディメンションに`data_owner`と追加のメタデータ値を`+meta`構文を使用して割り当てる方法を示しています。同様の構文は、エンティティとメジャーにも使用できます。

<File name='dbt_project.yml'>

```yml
semantic-models:
  jaffle_shop:
    ...
    [dimensions](/docs/build/dimensions):
      - name: order_date
        config:
          meta:
            data_owner: "Finance team"
            used_in_reporting: true
```


</File>
</TabItem>
</Tabs>
</VersionBlock>
