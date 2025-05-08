---
title: "集合演算子"
---

### Unions

`--select` または `--exclude` フラグに、スペースで区切られた複数の引数を指定すると、それらすべての結合が選択されます。リソースが少なくとも 1 つのセレクターに含まれている場合、そのリソースは最終的なセットに含まれます。

snowplow_sessions、snowplow_sessions のすべての祖先、fct_orders、および fct_orders のすべての祖先を実行します。


  ```bash
dbt run --select "+snowplow_sessions +fct_orders"
  ```

### Intersections

`--select` と `--exclude` に複数の引数をカンマで区切り、間に空白を入れない場合、dbt はすべての引数を満たすリソースのみを選択します。

snowplow_sessions と fct_orders の共通の祖先をすべて実行します。


  ```bash
dbt run --select "+snowplow_sessions,+fct_orders"
```


stg_invoices と stg_accounts の共通の子孫をすべて実行します:


  ```bash
dbt run --select "stg_invoices+,stg_accounts+"
  ```


marts/finance サブディレクトリにあり、nightly のタグが付けられたモデルを実行します:


  ```bash
dbt run --select "marts.finance,tag:nightly"
```
