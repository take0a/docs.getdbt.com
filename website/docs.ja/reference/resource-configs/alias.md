---
resource_types: [models, seeds, snapshots, tests]
description: "リソースにエイリアスを設定すると、ファイル名を使用する代わりに、データベース内でカスタム名を付けることができます。"
datatype: string
intro_text: モデル、データ テスト、スナップショット、またはシードにカスタム エイリアスを指定し、データベース内でよりユーザーフレンドリな名前を付けます。
---


<Tabs>
<TabItem value="model" label="Models">

`dbt_project.yml` ファイル、`models/properties.yml` ファイル、または SQL ファイルの設定ブロックで、モデルのカスタムエイリアスを指定します。

例えば、`sales_total` を計算するモデルがあり、よりユーザーフレンドリーなエイリアスを設定したい場合は、次の例のようにエイリアスを設定できます。

`dbt_project.yml` ファイルでは、次の例のように、プロジェクトレベルで `sales_total` モデルのデフォルトのエイリアスを設定します。

<File name='dbt_project.yml'>

```yml
models:
  your_project:
    sales_total:
      +alias: sales_dashboard
```
</File>

以下は、集中管理された構成に役立つ `models/properties.yml` ファイルのメタデータの一部として `alias` を指定します:

<File name='models/properties.yml'>

```yml
version: 2

models:
  - name: sales_total
    config:
      alias: sales_dashboard
```
</File>

以下は、`models/sales_total.sql` ファイル内で `alias` を直接割り当てます:

<File name='models/sales_total.sql'>

```sql
{{ config(
    alias="sales_dashboard"
) }}
```
</File>

これにより、データベースにはデフォルトの `analytics.finance.sales_total` ではなく `analytics.finance.sales_dashboard` が返されます。

</TabItem>

<TabItem value="seeds" label="Seeds">

`dbt_project.yml` ファイルまたは `properties.yml` ファイルでシードのエイリアスを設定します。以下の例は、`product_categories` という名前のシードに `categories_data` というエイリアスを設定する方法を示しています。

プロジェクトレベルの `dbt_project.yml` ファイルでの設定：

<File name='dbt_project.yml'>

```yml
seeds:
  your_project:
    product_categories:
      +alias: categories_data
```
</File>

`seeds/properties.yml` ファイル内:

<File name='seeds/properties.yml'>

```yml
version: 2

seeds:
  - name: product_categories
    config:
      alias: categories_data
```
</File>

これにより、データベース内の `analytics.finance.categories_data` という名前が返されます。

次の2番目の例では、`seeds/country_codes.csv` にあるシードが `country_mappings` という名前の <Term id="table" /> として構築されます。

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    country_codes:
      +alias: country_mappings

```
</File>
</TabItem>

<TabItem value="snapshot" label="Snapshots">

`dbt_project.yml` ファイル、`snapshots/snapshot_name.yml` ファイル、または設定ブロックでスナップショットのエイリアスを設定します。

次の例は、`your_snapshot` という名前のスナップショットに `the_best_snapshot` というエイリアスを設定する方法を示しています。

プロジェクトレベルの `dbt_project.yml` ファイルでの設定：

<File name='dbt_project.yml'>

```yml
snapshots:
  your_project:
    your_snapshot:
      +alias: the_best_snapshot
```
</File>

`snapshots/snapshot_name.yml` ファイル内:

<File name='snapshots/snapshot_name.yml'>

```yml
version: 2

snapshots:
  - name: your_snapshot_name
    config:
      alias: the_best_snapshot
</File>

In `snapshots/your_snapshot.sql` file:

<File name='snapshots/your_snapshot.sql'>

```sql
{{ config(
    alias="the_best_snapshot"
) }}
```
</File>

これにより、データベース内の `analytics.finance.the_best_snapshot` へのスナップショットが構築されます。

</TabItem>

<TabItem value="test" label="Tests">

データテストのエイリアスは、`dbt_project.yml` ファイル、`properties.yml` ファイル、またはモデルファイル内の設定ブロックで設定します。

次の例は、`order_id` という名前の一意のデータテストに `unique_order_id_test` というエイリアスを設定して、特定のデータテストを識別する方法を示しています。

プロジェクトレベルの `dbt_project.yml` ファイルでの設定：

<File name='dbt_project.yml'>

```yml
tests:
  your_project:
    +alias: unique_order_id_test
```
</File>

In the `models/properties.yml` file:

<File name='models/properties.yml'>

```yml
models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique:
              alias: unique_order_id_test
```
</File>

In `tests/unique_order_id_test.sql` file:

<File name='tests/unique_order_id_test.sql'>

```sql
{{ config(
    alias="unique_order_id_test",
    severity="error"
) }}
```
</File>

[`store_failures_as`](/reference/resource-configs/store_failures_as) を使用すると、データベースに `analytics.dbt_test__audit.orders_order_id_unique_order_id_test` という名前が返されます。


</TabItem>
</Tabs>

## 定義

オプションで、[モデル](/docs/build/models)、[データテスト](/docs/build/data-tests)、[スナップショット](/docs/build/snapshots)、または[シード](/docs/build/seeds)のカスタムエイリアスを指定します。

dbt がデータベースにリレーション (<Term id="table" />/<Term id="view" />) を作成する場合、`{{ database }}.{{ schema }}.{{ identifier }}` という形式で作成します (例: `analytics.finance.payments`)。

dbt の標準的な動作は次のとおりです。
* カスタムエイリアスが指定されていない場合、リレーションの識別子はリソース名 (つまりファイル名) になります。
* カスタムエイリアスが指定されている場合、リレーションの識別子は `{{ alias }}` の値になります。

**注** [エフェメラルモデル](/docs/build/materializations)では、dbt は常に <Term id="cte" /> 識別子にプレフィックス `__dbt__cte__` を適用します。つまり、エフェメラルモデルにエイリアスが設定されている場合、その CTE 識別子は `__dbt__cte__{{ alias }}` になりますが、エイリアスが設定されていない場合は `__dbt__cte__{{ filename }}` になります。

dbt がリレーションの `identifier` を生成する方法を変更する方法の詳細については、[エイリアスの使用](/docs/build/custom-aliases) を参照してください。

