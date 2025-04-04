---
title: "中かっこをネストしない"
id: "dont-nest-your-curlies"
---

### Poetry

**中かっこをネストしない**

> dbt が早くエラーを起こした場合
>
> そして Jinja があなたを不機嫌にさせている場合
>
> Slack に投稿しないでください
>
> ちょっと立ち止まって
>
> 中かっこをネストしているかどうか確認してください。

### Jinja

dbt プロジェクトで jinja コードを記述する場合、式を互いにネストしたくなるかもしれません。次の例を見てみましょう:

```
  {{ dbt_utils.date_spine(
      datepart="day",
      start_date=[ USE JINJA HERE ]
      )
  }}
```

jinja 式を別の jinja 式の中にネストするには、必要なコード (中括弧なし) を式内に直接配置するだけです。

**正しい例**
ここでは、`var()` コンテキスト メソッドの戻り値が `date_spine` マクロの `start_date` 引数として提供されています。すばらしい!

```
  {{ dbt_utils.date_spine(
      datepart="day",
      start_date=var('start_date')
      )
  }}
```

**誤った例**
Jinja 式の中にいることを示すと (`{{` 構文を使用)、Jinja 式の中にそれ以上の中括弧は必要ありません。このコードは、`date_spine` マクロの `start_date` 引数として、リテラル文字列値 `"{{ var('start_date') }}"` を提供します。これはおそらく、実際に実行したい操作ではありません。

```
-- Do not do this! It will not work!

  {{ dbt_utils.date_spine(
      datepart="day",
      start_date="{{ var('start_date') }}"
      )
  }}
```

もう一つの例を挙げます:

```sql
{# Either of these work #}

{% set query_sql = 'select * from ' ~ ref('my_model') %}

{% set query_sql %}
select * from {{ ref('my_model') }}
{% endset %}

{# This does not #}
{% set query_sql = "select * from {{ ref('my_model')}}" %}

```

### 例外

このルールには例外が 1 つあります。フックでは、中括弧内の中括弧が許容されます (つまり、`on-run-start`、`on-run-end`、`pre-hook`、および `post-hook`)。

次のようなコードは有効であり、推奨されます:
```
{{ config(post_hook="grant select on {{ this }} to role bi_role") }}
```

では、なぜこの場合、中括弧内の中括弧が許可されるのでしょうか? ここでは、実際には、文字列リテラル `"grant select on {{ this }} ..."` をこのモデルの post-hook の構成値として保存する必要があります。この文字列はモデルの実行時に再レンダリングされ、結果として `grant select on "schema"."table"....` のような意味のある SQL 式がデータベースに対して実行されます。これらのフックは、上記のルールの特別な例外です。
