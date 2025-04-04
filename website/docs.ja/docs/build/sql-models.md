---
title: "SQL モデル"
description: "SQL models are the building blocks of your dbt project."
id: "sql-models"
---

## 関連リファレンスドキュメント
* [モデル構成](/reference/model-configs)
* [モデル プロパティ](/reference/model-properties)
* [`run` コマンド](/reference/commands/run)
* [`ref` 関数](/reference/dbt-jinja-functions/ref)

## はじめる

:::info 最初のモデルの構築

dbt を初めて使用する場合は、[クイックスタート ガイド](/guides)を読んで、モデルを使用して最初の dbt プロジェクトを構築することをお勧めします。

:::

dbt の Python 機能は、SQL モデルの機能の拡張です。dbt を初めて使用する場合は、次のページを読む前に、まずこのページを読むことをお勧めします: ["Python モデル"](/docs/build/python-models)


SQL モデルは `select` ステートメントです。モデルは `.sql` ファイル (通常は `models` ディレクトリ内) で定義されます:
- 各 `.sql` ファイルには 1 つのモデル / `select` ステートメントが含まれます
- モデル名はファイル名から継承され、モデルの _filename_ と一致する必要があります (大文字と小文字の区別を含む)。大文字と小文字が一致しないと、dbt が構成を正しく適用できず、[dbt Explorer](/docs/collaborate/explore-projects) のメタデータに影響する可能性があります。
- モデル名にはドットではなくアンダースコアを使用することを強くお勧めします。たとえば、`models/my.model.sql` ではなく `models/my_model.sql` を使用します。
- モデルは `models` ディレクトリ内のサブディレクトリにネストできます。

モデルの命名方法については、[dbt モデルのスタイル設定方法](/best-practices/how-we-style/1-how-we-style-our-dbt-models) を参照してください。

[`dbt run` コマンド](/reference/commands/run) を実行すると、dbt はこのモデルを `create view as` または `create table as` ステートメントでラップして <Term id="data-warehouse" /> 構築します。

たとえば、次の `customers` モデルを考えてみましょう:

<File name='models/customers.sql'>

```sql
with customer_orders as (
    select
        customer_id,
        min(order_date) as first_order_date,
        max(order_date) as most_recent_order_date,
        count(order_id) as number_of_orders

    from jaffle_shop.orders

    group by 1
)

select
    customers.customer_id,
    customers.first_name,
    customers.last_name,
    customer_orders.first_order_date,
    customer_orders.most_recent_order_date,
    coalesce(customer_orders.number_of_orders, 0) as number_of_orders

from jaffle_shop.customers

left join customer_orders using (customer_id)
```

</File>

`dbt run` を実行すると、dbt はこれをターゲット スキーマ内に `customers` という名前の _view_ として構築します。

```sql
create view dbt_alice.customers as (
    with customer_orders as (
        select
            customer_id,
            min(order_date) as first_order_date,
            max(order_date) as most_recent_order_date,
            count(order_id) as number_of_orders

        from jaffle_shop.orders

        group by 1
    )

    select
        customers.customer_id,
        customers.first_name,
        customers.last_name,
        customer_orders.first_order_date,
        customer_orders.most_recent_order_date,
        coalesce(customer_orders.number_of_orders, 0) as number_of_orders

    from jaffle_shop.customers

    left join customer_orders using (customer_id)
)
```

なぜ _view_ が `dbt_alice.customers` という名前なのでしょうか? デフォルトでは、dbt は次の処理を行います:
* モデルを <Term id="view">views</Term> として作成します
* 定義したターゲット スキーマでモデルを構築します
* ファイル名をデータベースのビューまたは <Term id="table" /> 名として使用します

_configurations_ を使用してこれらの動作を変更できます。これについては後で詳しく説明します。

### FAQs
<FAQ path="Runs/checking-logs" />
<FAQ path="Models/create-a-schema" />
<FAQ path="Models/run-downtime" />
<FAQ path="Troubleshooting/sql-errors" />
<FAQ path="Models/sql-dialect" />

## モデルの構成
構成は「モデル設定」であり、`dbt_project.yml` ファイルと `config` ブロックを使用してモデル ファイルで設定できます。構成の例には次のものがあります:

* モデルが使用する <Term id="materialization" /> を変更する - [materialization](/docs/build/materializations) によって、dbt がウェアハウスにモデルを作成するために使用する SQL が決まります。

* モデルを個別の [schemas](/docs/build/custom-schemas) にビルドします。

* モデルに [tags](/reference/resource-configs/tags) を適用します。

モデル構成の例を次に示します:

<File name='dbt_project.yml'>

```yaml
name: jaffle_shop
config-version: 2
...

models:
  jaffle_shop: # this matches the `name:`` config
    +materialized: view # this applies to all models in the current project
    marts:
      +materialized: table # this applies to all models in the `marts/` directory
      marketing:
        +schema: marketing # this applies to all models in the `marts/marketing/`` directory

```

</File>


<File name='models/customers.sql'>

```sql

{{ config(
    materialized="view",
    schema="marketing"
) }}

with customer_orders as ...

```

</File>

設定は階層的に適用されることに注意してください。サブディレクトリに適用された設定は、一般的な設定を上書きします。

設定の詳細については、[リファレンス ドキュメント](/reference/model-configs) を参照してください。

### FAQs
<FAQ path="Models/available-materializations" />
<FAQ path="Models/available-configurations" />


## モデル間の依存関係の構築
クエリ内のテーブル名の代わりに [`ref` 関数](/reference/dbt-jinja-functions/ref) を使用することで、モデル間の依存関係を構築できます。`ref` の引数として別のモデルの名前を使用します。

<Tabs
  defaultValue="model"
  values={[
    {label: 'Model', value: 'model'},
    {label: 'Compiled code in dev', value: 'dev'},
    {label: 'Compiled code in prod', value: 'prod'},
  ]}>
  <TabItem value="model">


  <File name='models/customers.sql'>

  ```sql
  with customers as (

      select * from {{ ref('stg_customers') }}

  ),

  orders as (

      select * from {{ ref('stg_orders') }}

  ),

  ...

  ```

  </File>


  </TabItem>

  <TabItem value="dev">

```sql
create view dbt_alice.customers as (
  with customers as (

      select * from dbt_alice.stg_customers

  ),

  orders as (

      select * from dbt_alice.stg_orders

  ),

  ...
)

...

```


  </TabItem>

  <TabItem value="prod">

```sql
create view analytics.customers as (
  with customers as (

      select * from analytics.stg_customers

  ),

  orders as (

      select * from analytics.stg_orders

  ),

  ...
)

...

  ```

  </TabItem>
</Tabs>


dbt は `ref` 関数を使用して次の操作を行います:
* 従属非巡回グラフ (DAG) を作成して、モデルを実行する順序を決定します。
<Lightbox src="/img/dbt-dag.png" title="The DAG for our dbt project" />

* 個別の環境を管理します - dbt は、`ref` 関数で指定されたモデルを <Term id="table" /> (またはビュー) のデータベース名に置き換えます。重要なのは、これが環境対応であることです - `dbt_alice` という名前のターゲット スキーマで dbt を実行している場合、同じスキーマ内の上流テーブルから選択されます。上のタブをチェックして、これが実際にどのように機能するかを確認してください。

さらに、`ref` 関数は、モデルを再利用して繰り返しコードを削減できるように、モジュール変換を記述することを推奨します。

## モデルのテストと文書化

モデルをドキュメント化してテストすることもできます。詳細については、[テスト](/docs/build/data-tests)と[ドキュメント](/docs/build/documentation)のセクションに進んでください。

## Additional FAQs
<FAQ path="Project/example-projects" alt_header="Are there any example dbt models?" />
<FAQ path="Models/configurable-model-path" />
<FAQ path="Models/model-custom-schemas" />
<FAQ path="Project/unique-resource-names" />
<FAQ path="Models/removing-deleted-models" />
<FAQ path="Project/structure-a-project" alt_header="As I create more models, how should I keep my project organized? What should I name my models?" />
<FAQ path="Models/insert-records" />
<FAQ path="Project/why-not-write-dml" />
<FAQ path="Models/specifying-column-types" />
