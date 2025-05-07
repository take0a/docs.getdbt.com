---
sidebar_label: "docs"
resource_types: models
description: "Docs - dbt の構成について詳しく知るには、この詳細なガイドをお読みください。"
datatype: "{dictionary}"
default_value: {show: true}
---

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Sources', value: 'sources', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
    { label: 'Analyses', value: 'analyses', },
    { label: 'Macros', value: 'macros', },
  ]
}>

<TabItem value="models">

`dbt_project.yml` で設定することで、複数のリソースの `docs` 動作を一括で設定できます。また、`properties.yaml` ファイルの `docs` 設定を使用して、特定のリソースのドキュメント動作を設定または上書きすることもできます:


<File name='dbt_project.yml'>

```yml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    +docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")

```

</File>

<File name='models/schema.yml'>

  ```yml
version: 2

models:
  - name: model_name
    docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
```
</File>

</TabItem>

<TabItem value="sources">

このプロパティはソースに対して実装されていません。

</TabItem>

<TabItem value="seeds">

`dbt_project.yml` を含む YAML ファイルで docs プロパティを使用できます。

<File name='dbt_project.yml'>

```yml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    +docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
```

</File>

<File name='seeds/schema.yml'>

```yml
version: 2

seeds:
  - name: seed_name
    docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
```
</File>

</TabItem>

<TabItem value="snapshots">

`dbt_project.yml` を含む YAML ファイルで docs プロパティを使用できます:

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")

```

</File>

<File name='snapshots/schema.yml'>

```yml
version: 2

snapshots:
  - name: snapshot_name
    docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
```
</File>

</TabItem>

<TabItem value="analyses">

docsプロパティはYAMLファイル（`dbt_project.yml`を除く）で使用できます。詳細については、[分析プロパティ](/reference/analysis-properties)を参照してください。


<File name='analysis/schema.yml'>

```yml
version: 2

analyses:
  - name: analysis_name
    docs:
      show: true | false
      node_color: color_id # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
```
</File>

</TabItem>

<TabItem value="macros">

docsプロパティはYAMLファイル（`dbt_project.yml`を除く）で使用できます。詳細については[マクロプロパティ](/reference/macro-properties)を参照してください。

<File name='macros/schema.yml'>

```yml
version: 2

macros:
  - name: macro_name
    docs:
      show: true | false
```
</File>

</TabItem>

</Tabs>

## 定義

`docs` プロパティは、モデルにドキュメント固有の設定を提供するために使用できます。`show` 属性をサポートしており、自動生成されるドキュメントウェブサイトにノードを表示するかどうかを制御し、また、モデル、シード、スナップショット、および分析に対して `node_color` 属性もサポートしています。その他のノードタイプはサポートされていません。

**注:** 非表示のモデルも dbt DAG ビジュアライゼーションには表示されますが、「hidden」として識別されます。

## デフォルト

`show` のデフォルト値は `true` です。

## 例

### モデルを非表示としてマークする

```yml
models:
  - name: sessions__tmp
    docs:
      show: false
```

### モデルのサブフォルダを非表示としてマークする

**注:** これにより、dbt パッケージも非表示になる場合があります。

<File name='dbt_project.yml'>

```yml
models:
  # hiding models within the staging subfolder
  tpch:
    staging:
      +materialized: view
      +docs:
        show: false
  
  # hiding a dbt package
  dbt_artifacts:
    +docs:
      show: false
```

</File>

## カスタムノードカラー

`docs` 属性は `node_color` をサポートしており、[dbt Docs](/docs/build/view-documentation) 内の DAG 内の一部のノードタイプの表示色をカスタマイズできます。以下のファイルでノードカラーを定義し、必要に応じてオーバーライドを適用できます。

- `node_color` 階層:
- `<example-sql-file.sql>` は `schema.yml` をオーバーライドし、`dbt_project.yml` はオーバーライドされます。

カスタマイズした色を適用して表示するには、`dbt docs generate` コマンドを実行または再実行する必要があります。

:::info dbt Explorer ではカスタムノードカラーは適用されません

カスタム `node_color` 属性は dbt Explorer では適用されません。代わりに、Explorer は [レンズ](/docs/collaborate/explore-projects#lenses) を提供します。これは <Term id="dag"/> のマップレイヤーです。レンズを使用すると、プロジェクトのコンテキストメタデータを大規模に理解し、特定のモデルまたはモデルのサブセットを区別するのに役立ちます。

:::

## 例

サブディレクトリ内でサポートされているモデルに、16進コードまたは単純な色名に基づいてカスタム `node_colors` を追加します。

![Example](../../../../website/static/img/node_color_example.png)

`marts/core/fct_orders.sql` の `node_color: red` は、`node_color: gold` の `dbt_project.yml` をオーバーライドします。

`marts/core/schema.yml` の `node_color: #000000` は、`dbt_project.yml` の `node_color: gold` をオーバーライドします。

<File name='dbt_project.yml'>

```yml
models:
  tpch:
    staging:
      +materialized: view
      +docs:
        node_color: "#cd7f32"

    marts:
      core:
        materialized: table
        +docs:
          node_color: "gold"
```

</File>

<File name='marts/core/schema.yml'>

```yml
models:
  - name: dim_customers
    description: Customer dimensions table
    docs:
      node_color: '#000000'
```

</File>

<File name='marts/core/fct_orders.sql'>

```sql
{{
    config(
        materialized = 'view',
        tags=['finance'],
        docs={'node_color': 'red'}
    )
}}

with orders as (
    
    select * from {{ ref('stg_tpch_orders') }} 

),
order_item as (
    
    select * from {{ ref('order_items') }}

),
order_item_summary as (

    select 
        order_key,
        sum(gross_item_sales_amount) as gross_item_sales_amount,
        sum(item_discount_amount) as item_discount_amount,
        sum(item_tax_amount) as item_tax_amount,
        sum(net_item_sales_amount) as net_item_sales_amount
    from order_item
    group by
        1
),
final as (

    select 

        orders.order_key, 
        orders.order_date,
        orders.customer_key,
        orders.status_code,
        orders.priority_code,
        orders.clerk_name,
        orders.ship_priority,
                
        1 as order_count,                
        order_item_summary.gross_item_sales_amount,
        order_item_summary.item_discount_amount,
        order_item_summary.item_tax_amount,
        order_item_summary.net_item_sales_amount
    from
        orders
        inner join order_item_summary
            on orders.order_key = order_item_summary.order_key
)
select 
    *
from
    final

order by
    order_date

```

</File>

`node_color` が dbt ドキュメントと互換性がない場合は、次の例のようにコンパイル エラーが表示されます。

```shell
Invalid color name for docs.node_color: aweioohafio23f. It is neither a valid HTML color name nor a valid HEX code.
```

<File name='dbt_project.yml'>

```yml
models:
  tpch:
    marts:
      core:
        materialized: table
        +docs:
          node_color: "aweioohafio23f"
```

</File>
