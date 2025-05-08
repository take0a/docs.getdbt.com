---
title: "ソース内の引用の設定"
sidebar_label: "quoting"
datatype: boolean # -ish, it's actually a dictionary of bools
default: true
---
<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: jaffle_shop
    quoting:
      database: true | false
      schema: true | false
      identifier: true | false
    tables:
      - name: orders
        quoting:
          database: true | false
          schema: true | false
          identifier: true | false

```

</File>

## 定義

`{{ source() }}` 関数を直接リレーション参照に解決する際に、dbt がデータベース、スキーマ、および識別子を引用符で囲むかどうかをオプションで設定します。

この設定は、ソース内のすべてのテーブル、または特定のソース <Term id="table" /> に対して指定できます。特定のソーステーブルに対して定義された引用符設定は、最上位ソースに対して指定された引用符設定をオーバーライドします。

:::info BigQuery 用語

BigQuery の引用符設定では、`database` と `schema` を使用する必要がありますが、これらの設定はそれぞれ `project` と `dataset` の名前に適用されます。

:::


## デフォルト

デフォルト値はデータベースによって異なります。

ほとんどのアダプタでは、引用符はデフォルトで _true_ に設定されています。

なぜでしょうか？ 引用符で囲まれた識別子と引用符で囲まれていない識別子のリレーションからの選択は、どちらも同じように簡単です。引用符を使用すると、それらの識別子に予約語や特殊文字を使用できますが、可能な限り使用を避けることをお勧めします。

Snowflake では、引用符はデフォルトで _false_ に設定されています。

引用符で囲まれた識別子を使用してリレーションを作成すると、それらの識別子の大文字と小文字が区別されます。そのため、それらの識別子からの選択は非常に困難になります。大文字と小文字が区別される、予約語、または特殊文字を含むリレーション識別子に対して引用符を再度有効にすることはできますが、可能な限り使用を避けることをお勧めします。

## 例

<File name='models/<filename>.yml'>

```yaml
version: 2

sources:
  - name: jaffle_shop
    database: raw
    quoting:
      database: true
      schema: true
      identifier: true

    tables:
      - name: orders
      - name: customers
        # This overrides the `jaffle_shop` quoting config
        quoting:
          identifier: false


```

</File>

ダウンストリームモデルの場合:

<File name='models/<filename>.yml'>

```sql
select
  ...

-- this should be quoted
from {{ source('jaffle_shop', 'orders') }}

-- here, the identifier should be unquoted
left join {{ source('jaffle_shop', 'customers') }} using (order_id)

```

</File>


これは次のようにコンパイルされます:

```sql
select
  ...

-- this should be quoted
from "raw"."jaffle_shop"."orders"

-- here, the identifier should be unquoted
left join "raw"."jaffle_shop".customers using (order_id)

```
