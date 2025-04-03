---
title: SQL のスタイル設定方法
id: 2-how-we-style-our-sql
---

## Basics

- ☁️ これらのスタイル ルールを自動的に維持するには、[SQLFluff](https://sqlfluff.com/) を使用します。
  - `.sqlfluff` 構成ファイルをニーズに合わせてカスタマイズします。
  - 私たちのプロジェクトで使用しているルールについては、[SQLFluff 構成ファイル](https://github.com/dbt-labs/jaffle-shop-template/blob/main/.sqlfluff)を参照してください。
  - 標準の `.sqlfluffignore` ファイルを使用してファイルとディレクトリを除外します。構文の詳細については、[.sqlfluffignore 構文ドキュメント](https://docs.sqlfluff.com/en/stable/configuration/index.html) を参照してください。
    - 不要なフォルダーとファイル (`target/`、`dbt_packages/`、`macros/` など) を除外すると、リンティングが高速化され、実行時間が短縮され、無関係なログを回避するのに役立ちます。
- 👻 コンパイルされた SQL に含めるべきではないコメントには、Jinja コメント (`{# #}`) を使用します。
- ⏭️ 末尾にカンマを使用します。
- 4️⃣ インデントは 4 スペースにする必要があります。
- 📏 SQL 行の長さは 80 文字以下にする必要があります。
- ⬇️ フィールド名、キーワード、関数名はすべて小文字にする必要があります。
- 🫧 フィールドまたはテーブルにエイリアスを付ける場合は、`as` キーワードを明示的に使用する必要があります。

:::info
☁️ dbt Cloud ユーザーは、組み込みの [SQLFluff Cloud IDE 統合](https://docs.getdbt.com/docs/cloud/dbt-cloud-ide/lint-format) を使用して、SQL を自動的に lint およびフォーマットできます。デフォルトのスタイル シートは、このガイドで説明されている dbt Labs スタイルに基づいていますが、ニーズに合わせてカスタマイズできます。外部ツールを設定する必要はなく、`Lint` を押すだけです。また、そのスタイルが好みであれば、より独自の [sqlfmt](http://sqlfmt.com/) フォーマッタも利用できます。
:::

## フィールド、集計、グループ化

- 🔙 フィールドは、集計関数やウィンドウ関数の前に指定する必要があります。
- 🤏🏻 パフォーマンスを向上させるには、別のテーブルに結合する前に、できるだけ早く (可能な限り小さいデータ セットで) 集計を実行する必要があります。
- 🔢 番号による順序付けとグループ化 (例: group by 1, 2) は、列名をリストするよりも優先されます (理由については、[この古典的な暴言](https://www.getdbt.com/blog/write-better-sql-a-defense-of-group-by-1) を参照してください)。複数の列でグループ化する場合は、モデル設計を再検討する価値があることに注意してください。

## 結合

- 👭🏻 重複を明示的に削除する場合を除き、`union` よりも `union all` を使用することをお勧めします。
- 👭🏻 2 つ以上のテーブルを結合する場合は、必ず列名の前にテーブル名を付けます。1 つのテーブルからのみ選択する場合は、プレフィックスは必要ありません。
- 👭🏻 結合タイプを明示的に指定します (つまり、`join` ではなく `inner join` と記述します)。
- 🥸 結合条件ではテーブル エイリアス (特に頭字語) を使用しないでください。"customers" と比較すると、"c" というテーブルが何であるかを理解するのが難しくなります。
- ➡️ 結合がわかりやすくなるように、常に左から右へ移動します。`右結合` は、多くの場合、`選択元` のテーブルと `結合` 先のテーブルを変更する必要があることを示します。

## 「インポート」CTE

- 🔝 すべての `{{ ref('...') }}` ステートメントは、ファイルの先頭の CTE に配置する必要があります。
- 📦 「インポート」CTE は、参照先のテーブルにちなんで命名する必要があります。
- 🤏🏻 CTE によってスキャンされるデータを可能な限り制限します。可能な場合は、実際に使用する列のみを選択し、`where` 句を使用して不要なデータを除外します。
- 例えば：

```sql
with

orders as (

    select
        order_id,
        customer_id,
        order_total,
        order_date

    from {{ ref('orders') }}

    where order_date >= '2020-01-01'

)
```

## 「ファンクショナル」CTE

- ☝🏻 パフォーマンスが許す限り、CTE は単一の論理的な作業単位を実行する必要があります。
- 📖 CTE 名は、その機能を伝えるために必要なだけ冗長にする必要があります。たとえば、`user_events` ではなく `events_joined_to_users` です (これは適切なモデル名ですが、特定の機能や変換を説明するものではありません)。
- 🌉 モデル間で重複している CTE は、独自の中間モデルに取り出す必要があります。独自のモデルにリファクタリングする必要がある、繰り返しロジックのチャンクに注意してください。
- 🔚 モデルの最後の行は、最終出力 CTE からの `select *` である必要があります。これにより、モデルの開発中に、モデル内のさまざまなステップからの出力を簡単に実現および監査できます。そのステップからの出力を表示するには、`select` ステートメントで参照されている CTE を変更するだけです。

## モデル構成

- 📝 モデル固有の属性 (sort/dist キーなど) はモデル内で指定する必要があります。
- 📂 特定の構成がディレクトリ内のすべてのモデルに適用される場合、`dbt_project.yml` ファイルで指定する必要があります。
- 👓 読みやすさを最大限に高めるには、モデル内の構成を次のように指定する必要があります:

```sql
{{
    config(
      materialized = 'table',
      sort = 'id',
      dist = 'id'
    )
}}
```

## Example SQL

```sql
with

events as (

    ...

),

{# CTE comments go here #}
filtered_events as (

    ...

)

select * from filtered_events
```

### Example SQL

```sql
with

my_data as (

    select
        field_1,
        field_2,
        field_3,
        cancellation_date,
        expiration_date,
        start_date

    from {{ ref('my_data') }}

),

some_cte as (

    select
        id,
        field_4,
        field_5

    from {{ ref('some_cte') }}

),

some_cte_agg as (

    select
        id,
        sum(field_4) as total_field_4,
        max(field_5) as max_field_5

    from some_cte

    group by 1

),

joined as (

    select
        my_data.field_1,
        my_data.field_2,
        my_data.field_3,

        -- use line breaks to visually separate calculations into blocks
        case
            when my_data.cancellation_date is null
                and my_data.expiration_date is not null
                then expiration_date
            when my_data.cancellation_date is null
                then my_data.start_date + 7
            else my_data.cancellation_date
        end as cancellation_date,

        some_cte_agg.total_field_4,
        some_cte_agg.max_field_5

    from my_data

    left join some_cte_agg
        on my_data.id = some_cte_agg.id

    where my_data.field_1 = 'abc' and
        (
            my_data.field_2 = 'def' or
            my_data.field_2 = 'ghi'
        )

    having count(*) > 1

)

select * from joined
```
