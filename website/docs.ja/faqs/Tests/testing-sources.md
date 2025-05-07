---
title: ソースのみでテストを実行するにはどうすればよいですか?
description: "ソースをテストするには、select source コマンドを使用します。"
sidebar_label: 'すべてのソースでテストを実行する'
id: testing-sources

---

すべてのソースに対してテストを実行するには、次のコマンドを使用します:

```shell
  dbt test --select "source:*"
```

(`--select` の代わりに `-s` ショートカットを使用することもできます)

1 つのソース (およびそのすべてのテーブル) に対してテストを実行するには:

```shell
$ dbt test --select source:jaffle_shop
```

また、1 つのソース <Term id="table" /> のみでテストを実行するには、次のようにします:

```shell
$ dbt test --select source:jaffle_shop.orders
```

