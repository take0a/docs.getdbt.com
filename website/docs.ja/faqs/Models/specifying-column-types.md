---
title: 列の種類を指定するにはどうすればよいですか?
description: "モデル内の列タイプを指定する"
sidebar_label: 'モデル内の列タイプを指定する'
id: specifying-column-types

---
モデル内の列を正しい型にキャストするだけです:

```sql
select
    id,
    created::timestamp as created
from some_other_table
```

次のようなステートメントを実行することに慣れている場合は、次のような疑問が生じるかもしれません:

```sql
create table dbt_alice.my_table
  id integer,
  created timestamp;

insert into dbt_alice.my_table (
  select id, created from some_other_table
)
```

比較すると、dbt は `create table as` ステートメントを使用してこの <Term id="table" /> を構築します:

```sql
create table dbt_alice.my_table as (
  select id, created from some_other_table
)
```

モデルクエリが正しい列タイプを返す限り、作成するテーブルも正しい列タイプになります。

追加の列オプションを定義するには、次の手順に従ってください。

* 列に一意性と非NULL制約を適用する代わりに、dbt の [データテスト](/docs/build/data-tests) 機能を使用して、モデルに関するアサーションが真であることを確認します。
* 列のデフォルト値を作成する代わりに、SQL を使用してデフォルト値を指定します (例: `coalesce(updated_at, current_timestamp()) as updated_at`)。
* 列を変更する必要があるエッジケース (例: Redshift での列レベルのエンコード) では、[post-hook](/reference/resource-configs/pre-hook-post-hook) を使用して実装することを検討してください。
