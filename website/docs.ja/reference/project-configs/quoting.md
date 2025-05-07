---
title: "プロジェクトのクォートの設定"
sidebar_label: "quoting"
datatype: boolean # -ish, it's actually a dictionary of bools
description: "dbt でのクォート設定を理解するには、このガイドをお読みください。"
default: true
---
<File name='dbt_project.yml'>

```yml
quoting:
  database: true | false
  schema: true | false
  identifier: true | false

```

</File>

## 定義
以下の場合に、dbt がデータベース、スキーマ、および識別子を引用符で囲むかどうかをオプションで設定します。
* リレーション（テーブル/ビュー）の作成
* `ref` 関数を直接リレーション参照に解決する

:::info BigQuery 用語

BigQuery の引用符設定では、`database` と `schema` を使用する必要がありますが、これらの設定はそれぞれ `project` と `dataset` の名前に適用されます。
:::

## デフォルト

デフォルト値はデータベースによって異なります。

<Tabs
  defaultValue="default"
  values={[
    { label: 'Default', value: 'default', },
    { label: 'Snowflake', value: 'snowflake', },
  ]
}>
<TabItem value="default">

ほとんどのアダプタでは、引用符はデフォルトで `true` に設定されています。

なぜでしょうか？ 引用符で囲まれた識別子でも、囲まれていない識別子でも、リレーションからの選択は同じように簡単です。引用符で囲むことで、識別子内で予約語や特殊文字を使用できますが、可能な限り引用符の使用は避けることをお勧めします。

  <File name='dbt_project.yml'>

```yml
quoting:
  database: true
  schema: true
  identifier: true

```

</File>
</TabItem>
<TabItem value="snowflake">

Snowflakeでは、引用符はデフォルトで `false` に設定されています。

引用符で囲まれた識別子を使用してリレーションを作成すると、それらの識別子の大文字と小文字が区別されます。そのため、それらの識別子からの選択は非常に困難になります。大文字と小文字が区別される、予約語を含む、または特殊文字を含むリレーション識別子に対して引用符を再度有効にすることは可能ですが、可能な限り避けることをお勧めします。

<File name='dbt_project.yml'>

```yml
quoting:
  database: false
  schema: false
  identifier: false

```

</File>


</TabItem>

</Tabs>

## 例
プロジェクトの引用符を `false` に設定する:

<File name='dbt_project.yml'>

```yml
quoting:
  database: false
  schema: false
  identifier: false

```

dbt は引用符なしのリレーションを作成します:

```sql
create table analytics.dbt_alice.dim_customers
```

</File>


## おすすめ

### Snowflake

すべての引用符設定を `False` に設定してください。これは、予約語を識別子として使用できないことを意味します。ただし、通常はこれらの予約語の使用を避けることをお勧めします。

Snowflake のソーステーブルで引用符で囲まれたデータベース、スキーマ、またはテーブル識別子を使用している場合は、source.yml ファイルで設定できます。[詳細については、引用符の設定を参照してください](/reference/resource-properties/quoting)。



#### 説明:

ほとんどのデータベースでは引用符で囲まれていない識別子は小文字で変換されますが、Snowflakeでは引用符で囲まれていない識別子は大文字で変換されます。モデル名が小文字で引用符で囲まれている場合は、引用符なしで参照することはできません。詳細については、以下の例をご覧ください。

<File name='snowflake_casing.sql'>

```sql
/*
    You can run the following queries against your database
    to build an intuition for how quoting works on Snowflake.
*/

-- This is the output of an example `orders.sql` model with quoting enabled
create table "analytics"."orders" as (

  select 1 as id

);

/*
    These queries WILL NOT work! Since the table above was created with quotes,
    Snowflake created the orders table with a lowercase schema and identifier.

    Since unquoted identifiers are automatically uppercased, both of the
    following queries are equivalent, and neither will work correctly.
*/

select * from analytics.orders;
select * from ANALYTICS.ORDERS;

/*
    To query this table, you'll need to quote the schema and table. This
    query should indeed complete without error.
*/

select * from "analytics"."orders";


/*
    To avoid this quoting madness, you can disable quoting for schemas
    and identifiers in your dbt_project.yml file. This means that you
    won't be able to use reserved words as model names, but you probably
    shouldn't be doing that anyway! Assuming schema and identifier quoting is
    disabled, the following query would indeed work:
*/

select * from analytics.orders;
```

</File>



### その他のデータウェアハウス
データウェアハウスのデフォルト値はそのままにしておきます。
