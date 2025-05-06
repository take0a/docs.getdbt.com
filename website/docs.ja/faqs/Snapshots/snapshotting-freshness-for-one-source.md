---
title: 1 つのソースのみの最新スナップショットを取得するにはどうすればよいですか?
description: "特定のソースのスナップショットを作成するには select フラグを使用します"
sidebar_label: '特定のソースのスナップショットの最新化'
id: snapshotting-freshness-for-one-source

---


特定のソースのスナップショットの最新版を取得するには、`--select` フラグを使用します。例:

```shell
# Snapshot freshness for all Jaffle Shop tables:
$ dbt source freshness --select source:jaffle_shop

# Snapshot freshness for a particular source <Term id="table" />:
$ dbt source freshness --select source:jaffle_shop.orders

# Snapshot freshness for multiple particular source tables:
$ dbt source freshness --select source:jaffle_shop.orders source:jaffle_shop.customers
```

詳細については、[`source freshness` コマンド リファレンス](/reference/commands/source)を参照してください。
