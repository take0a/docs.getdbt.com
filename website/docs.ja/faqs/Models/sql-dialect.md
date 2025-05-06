---
title: どの SQL 方言でモデルを記述すればよいですか? または、dbt はどの SQL 方言を使用していますか?
description: "データベースに独自のSQL方言を使用する"
sidebar_label: 'どの SQL 方言を使用すればよいですか?'
id: sql-dialect
---

dbt は魔法のように思えるかもしれませんが、実際には魔法ではありません。内部的には、独自のウェアハウス内で SQL を実行しているため、データがウェアハウスの外部で処理されることはありません。

そのため、モデルでは **独自のデータベースの SQL 方言** を使用する必要があります。その後、dbt が `select` ステートメントを適切な <Term id="ddl" /> または <Term id="dml" /> でラップすると、ウェアハウスに適した DML が使用されます。このロジックはすべて dbt に記述されています。

dbt がサポートするデータベース、プラットフォーム、クエリエンジンの詳細については、[サポートされるデータプラットフォーム](/docs/supported-data-platforms) ドキュメントをご覧ください。

この仕組みについてもう少し詳しく知りたいですか？各ウェアハウスで動作する SQL スニペットを検討してください:

<File name='models/test_model.sql'>

```sql
select 1 as my_column

```

</File>

既存の <Term id="table" /> を置き換えるには、異なるウェアハウスで実行される SQL dbt の _説明的な_ 例を次に示します (実際の SQL はこれよりもはるかに複雑になる可能性があります)。

<Tabs
  defaultValue="redshift"
  values={[
    {label: 'Redshift', value: 'redshift'},
    {label: 'BigQuery', value: 'bigquery'},
    {label: 'Snowflake', value: 'snowflake'},
  ]}>
  <TabItem value="redshift">

```sql
-- you can't create or replace on redshift, so use a transaction to do this in an atomic way

begin;

create table "dbt_alice"."test_model__dbt_tmp" as (
    select 1 as my_column
);

alter table "dbt_alice"."test_model" rename to "test_model__dbt_backup";

alter table "dbt_alice"."test_model__dbt_tmp" rename to "test_model"

commit;

begin;

drop table if exists "dbt_alice"."test_model__dbt_backup" cascade;

commit;
```

  </TabItem>

  <TabItem value="bigquery">

```sql

-- Make an API call to create a dataset (no DDL interface for this)!!;

create or replace table `dbt-dev-87681`.`dbt_alice`.`test_model` as (
  select 1 as my_column
);
```

  </TabItem>

  <TabItem value="snowflake">

```sql
create schema if not exists analytics.dbt_alice;

create or replace table analytics.dbt_alice.test_model as (
    select 1 as my_column
);
```

  </TabItem>
</Tabs>
