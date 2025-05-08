---
title: "まとめると"
---


  ```bash
dbt run --select "my_package.*+"      # select all models in my_package and their children
dbt run --select "+some_model+"       # select some_model and all parents and children

dbt run --select "tag:nightly+"      # select "nightly" models and all children
dbt run --select "+tag:nightly+"      # select "nightly" models and all parents and children

dbt run --select "@source:snowplow"   # build all models that select from snowplow sources, plus their parents

dbt test --select "config.incremental_strategy:insert_overwrite,test_name:unique"   # execute all `unique` tests that select from models using the `insert_overwrite` incremental strategy
```



これはかなり複雑になります！例えば、snowplow のデータとフィードエクスポートから構築されたモデルを毎晩実行し、最大の増分モデル（と、さらにもう1つのモデル）を除外するとします。


  ```bash
dbt run --select "@source:snowplow,tag:nightly models/export" --exclude "package:snowplow,config.materialized:incremental export_performance_timing"
```


このコマンドは、以下の条件を満たすすべてのモデルを選択します。
* snowplow ソースとその親から選択し、かつ「nightly」タグが付けられているモデル
* `export` モデルサブフォルダで定義されているモデル

以下の条件を満たすモデルは除きます。
* snowplow パッケージで定義され、増分的にマテリアライズされているモデル
* `export_performance_timing` という名前が付けられているモデル