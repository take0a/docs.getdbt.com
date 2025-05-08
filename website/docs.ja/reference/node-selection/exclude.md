---
title: "実行からモデルを除外する"
sidebar_label: "Exclude"
---

### モデルの除外

dbt は、`--select` と同じ意味を持つ `--exclude` フラグを提供します。`--exclude` フラグで指定されたモデルは、`--select` で選択されたモデルセットから除外されます。

```bash
dbt run --select "my_package".*+ --exclude "my_package.a_big_model+"    # select all models in my_package and their children except a_big_model and its children
```

名前または系統によって特定のリソースを除外します:

```bash
# test
dbt test --exclude "not_null_orders_order_id"   # test all models except the not_null_orders_order_id test
dbt test --exclude "orders"                     # test all models except tests associated with the orders model

# seed
dbt seed --exclude "account_parent_mappings"    # load all seeds except account_parent_mappings

# snapshot
dbt snapshot --exclude "snap_order_statuses"    # execute all snapshots except snap_order_statuses
```
