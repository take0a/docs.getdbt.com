---
title: "DAGにソースを追加する"
sidebar_label: "Sources"
description: "dbt で開発するときにデータ ソース テーブルを定義します。"
id: "sources"
search_weight: "heavy"
---

## 関連リファレンスドキュメント
* [ソースプロパティ](/reference/source-properties)
* [ソース構成](/reference/source-configs)
* [`{{ source() }}` Jinja関数](/reference/dbt-jinja-functions/source)
* [`source freshness` コマンド](/reference/commands/source)

## ソースの使用
ソースを使用すると、抽出ツールとロードツールによってウェアハウスにロードされたデータに名前を付け、説明を付けることができます。
dbtでこれらのテーブルをソースとして宣言することで、次のことが可能になります。
- [`{{ source() }}` 関数](/reference/dbt-jinja-functions/source)を使用して、モデル内のソーステーブルから選択し、データの系統を定義する
- ソースデータに関する仮定をテストする
- ソースデータの鮮度を計算する

### ソースの宣言

ソースは、`sources:` キーの下にネストされた `.yml` ファイルで定義されます。

<File name='models/<filename>.yml'>

```yaml
version: 2

sources:
  - name: jaffle_shop
    database: raw  
    schema: jaffle_shop  
    tables:
      - name: orders
      - name: customers

  - name: stripe
    tables:
      - name: payments
```

</File>

*デフォルトでは、`schema` は `name` と同じになります。
既存のスキーマとは異なるソース名を使用する場合にのみ、`schema` を追加してください。

これらのファイルについてまだよく知らない場合は、続行する前に [properties.yml ファイルに関するドキュメント](/reference/configs-and-properties) を確認してください。

### ソースからの選択

ソースを定義すると、[`{{ source()}}` 関数](/reference/dbt-jinja-functions/source) を使用してモデルから参照できるようになります。


<File name='models/orders.sql'>

```sql
select
  ...

from {{ source('jaffle_shop', 'orders') }}

left join {{ source('jaffle_shop', 'customers') }} using (customer_id)

```

</File>

dbt will compile this to the full <Term id="table" /> name:

<File name='target/compiled/jaffle_shop/models/my_model.sql'>

```sql

select
  ...

from raw.jaffle_shop.orders

left join raw.jaffle_shop.customers using (customer_id)

```

</File>

`{{ source () }}` 関数を使用すると、モデルとソース テーブルの間に依存関係も作成されます。

<Lightbox src="/img/docs/building-a-dbt-project/sources-dag.png" title="The source function tells dbt a model is dependent on a source "/>

### ソースのテストとドキュメント作成
以下の操作も可能です。
- ソースにデータテストを追加する
- ソースに説明を追加する（ドキュメントサイトの一部としてレンダリングされる）

既にモデルにテストと説明を追加している場合は、これらの概念は既にお馴染みのはずです（まだ追加していない場合は、[テスト](/docs/build/data-tests) と [ドキュメント](/docs/build/documentation) のガイドをご覧ください）。

<File name='models/<filename>.yml'>

```yaml
version: 2

sources:
  - name: jaffle_shop
    description: This is a replica of the Postgres database used by our app
    tables:
      - name: orders
        description: >
          One record per order. Includes cancelled and deleted orders.
        columns:
          - name: id
            description: Primary key of the orders table
            tests:
              - unique
              - not_null
          - name: status
            description: Note that the status can change over time

      - name: ...

  - name: ...
```

</File>

ソースで使用可能なプロパティの詳細については、[リファレンス セクション](/reference/source-properties) を参照してください。

### FAQs
<FAQ path="Project/source-has-bad-name" />
<FAQ path="Project/source-in-different-database" />
<FAQ path="Models/source-quotes" />
<FAQ path="Tests/testing-sources" />
<FAQ path="Runs/running-models-downstream-of-source" />

## ソースデータの鮮度
dbt は、いくつかの追加設定を行うことで、ソーステーブル内のデータの鮮度をオプションで取得できます。
これは、データパイプラインが健全な状態にあるかどうかを把握するのに役立ち、ウェアハウスの SLA を定義する上で重要な要素となります。

### ソースの鮮度情報の宣言
ソースの鮮度情報を設定するには、ソースに「freshness」ブロックを追加し、テーブル宣言に「loaded_at_field」を追加します:

<File name='models/<filename>.yml'>

```yaml
version: 2

sources:
  - name: jaffle_shop
    database: raw
    freshness: # default freshness
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    loaded_at_field: _etl_loaded_at

    tables:
      - name: orders
        freshness: # make this a little more strict
          warn_after: {count: 6, period: hour}
          error_after: {count: 12, period: hour}

      - name: customers # this inherits the default freshness defined in the jaffle_shop source block at the beginning


      - name: product_skus
        freshness: null # do not check freshness for this table
```

</File>

`freshness` ブロックでは、`warn_after` と `error_after` のいずれか、または両方を指定できます。
どちらも指定されていない場合、dbt はこのソース内のテーブルの鮮度を計算しません。

さらに、テーブルの鮮度を計算するには `loaded_at_field` が必要です。
`loaded_at_field` が指定されていない場合、dbt はテーブルの鮮度を計算しません。

これらの設定は階層的に適用されるため、`source` に指定された `freshness` と `loaded_at_field` の値は、そのソースで定義されているすべての `tables` に適用されます。
これは、ソース内のすべてのテーブルで同じ `loaded_at_field` が使用されている場合に便利です。この設定は、最上位レベルのソース定義で 1 回だけ指定すればよいためです。

### ソースの鮮度を確認する
ソースの鮮度情報を取得するには、「dbt source freshness」コマンドを使用します（[リファレンスドキュメント](/reference/commands/source)）。

```
$ dbt source freshness
```

dbt はバックグラウンドで、freshness プロパティを使用して、以下に示す `select` クエリを構築します。
このクエリは [クエリ ログ](/faqs/Runs/checking-logs) で確認できます。

```sql
select
  max(_etl_loaded_at) as max_loaded_at,
  convert_timezone('UTC', current_timestamp()) as calculated_at
from raw.jaffle_shop.orders

```

このクエリの結果は、ソースが新鮮かどうかを判断するために使用されます:

<Lightbox src="/img/docs/building-a-dbt-project/snapshot-freshness.png" title="Uh oh! Not everything is as fresh as we'd like!"/>

### ソースの鮮度に基づいてモデルを構築する

ベストプラクティスとして、[データソースの鮮度](/docs/build/sources#declaring-source-freshness)を使用することを推奨します。
これにより、設定を `.yml` ファイルに転送し、ソースの鮮度を[モデルレベル](/reference/resource-properties/freshness)で定義できるようになります。

dbt でソースの鮮度に基づいてモデルを構築するには、以下の手順に従います:

1. `dbt source freshness` を実行して、ソースの鮮度を確認します。
2. `dbt build --select source_status:fresher+` コマンドを使用して、より新鮮なソースの下流でモデルを構築およびテストします。

これらのコマンドを順番に使用することで、モデルが最新のデータで更新されることが保証されます。
これにより、変更されていないデータに対する無駄な計算サイクルが排除され、必要な場合にのみモデルが構築されます。

[ソース鮮度スナップショット](/docs/deploy/source-freshness#enabling-source-freshness-snapshots)を30分に設定してソースの鮮度を確認し、1時間ごとに再構築するジョブを実行してモデルを再構築します。
この設定により、すべてのモデルが取得され、ソース鮮度が期限切れになった場合に1回の試行で再構築されます。
詳細については、[ソース鮮度スナップショットの頻度](/docs/deploy/source-freshness#source-freshness-snapshot-frequency)を参照してください。

### フィルター

一部のデータベースでは、テーブル全体のスキャン（コストのかかる可能性がある）を回避するために、特定の列に対するフィルター処理が必要なテーブルが存在する場合があります。
このようなテーブルで最新性チェックを行うには、設定に「filter」引数を追加します。例：`filter: _etl_loaded_at >= date_sub(current_date(), interval 1 day)`。
上記の例では、結果のクエリは次のようになります。

```sql
select
  max(_etl_loaded_at) as max_loaded_at,
  convert_timezone('UTC', current_timestamp()) as calculated_at
from raw.jaffle_shop.orders
where _etl_loaded_at >= date_sub(current_date(), interval 1 day)
```

### FAQs
<FAQ path="Project/exclude-table-from-freshness" />
<FAQ path="Snapshots/snapshotting-freshness-for-one-source" />
<FAQ path="Project/dbt-source-freshness" />