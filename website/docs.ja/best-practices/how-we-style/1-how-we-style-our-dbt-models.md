---
title: dbt モデルのスタイル設定方法
id: 1-how-we-style-our-dbt-models
---

## フィールドとモデル名

- 👥 モデルは、`customers`、`orders`、`products` のように複数形にする必要があります。
- 🔑 各モデルには主キーが必要です。
- 🔑 モデルの主キーは、`account_id` のように、`<object>_id` という名前にする必要があります。これにより、下流の結合モデルで参照されている `id` がわかりやすくなります。
- dbt モデルの命名にはアンダースコアを使用し、ドットは使用しないでください。
  - ✅  `models_without_dots`
  - ❌ `models.with.dots`
  - ほとんどのデータ プラットフォームでは、`database.schema.object` を区切るためにドットが使用されるため、ドットの代わりにアンダースコアを使用すると、[引用符](/reference/resource-properties/quoting)の必要性が減り、dbt Cloud の特定の部分で問題が発生するリスクも減ります。詳細については、[この GitHub の問題](https://github.com/dbt-labs/dbt-core/issues/3246) を参照してください。
- 🔑 キーは文字列データ型である必要があります。
- 🔑 一貫性が重要です。可能な場合は、モデル間で同じフィールド名を使用します。たとえば、`customers` テーブルのキーの名前は、`user_id` や 'id' ではなく、`customer_id` にする必要があります。
- ❌ 略語や別名は使用しないでください。簡潔さよりも読みやすさを重視してください。たとえば、`customer` の代わりに `cust` を使用したり、`orders` の代わりに `o` を使用したりしないでください。
- ❌ 列名に予約語を使用しないでください。
- ➕ ブール値には `is_` または `has_` というプレフィックスを付ける必要があります。
- 🕰️ タイムスタンプ列の名前は `<event>_at` (たとえば `created_at`) とし、UTC で指定する必要があります。別のタイムゾーンを使用する場合は、サフィックス (`created_at_pt`) で示す必要があります。
- 📆 日付の名前は `<event>_date` にする必要があります。たとえば、`created_date.` です。
- 🔙 イベントの日付と時刻は過去形でなければなりません（「作成済み」、「更新済み」、または「削除済み」）。
- 💱 価格/収益フィールドは、10 進通貨で入力する必要があります ($19.99 の場合は `19.99`。多くのアプリ データベースでは、価格がセント単位の整数として保存されます)。10 進数以外の通貨を使用する場合は、接尾辞 (`price_in_cents`) でこれを示します。
- 🐍 スキーマ、テーブル、列の名前は `snake_case` にする必要があります。
- 🏦 ソース用語ではなく、_ビジネス_ 用語に基づいた名前を使用します。たとえば、ソース データベースでは `user_id` が使用され、ビジネスでは `customer_id` と呼ばれている場合は、モデルでは `customer_id` を使用します。
- 🔢 モデルのバージョンでは、一貫性を保つために、サフィックス `_v1`、`_v2` などを使用する必要があります (`customers_v1` および `customers_v2`)。
- 🗄️ データ型の一貫した順序を使用し、以下の例のように、列を型別にグループ化してラベル付けすることを検討してください。これにより、結合エラーが最小限に抑えられ、モデルが読みやすくなるだけでなく、下流のデータ コンシューマーがデータ型を理解し、必要な列のモデルをスキャンしやすくなります。ID、文字列、数値、ブール値、日付、タイムスタンプの順序を使用することをお勧めします。

## Example model

```sql
with

source as (

    select * from {{ source('ecom', 'raw_orders') }}

),

renamed as (

    select

        ----------  ids
        id as order_id,
        store_id as location_id,
        customer as customer_id,

        ---------- strings
        status as order_status,

        ---------- numerics
        (order_total / 100.0)::float as order_total,
        (tax_paid / 100.0)::float as tax_paid,

        ---------- booleans
        is_fulfilled,

        ---------- dates
        date(order_date) as ordered_date,

        ---------- timestamps
        ordered_at

    from source

)

select * from renamed
```
