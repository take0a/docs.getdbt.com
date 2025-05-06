---
title: dbt 出力にマクロが多数含まれているのはなぜですか?
description: "dbt プロジェクトには多くのマクロが含まれています。"
sidebar_label: 'dbtプロジェクトには多くのマクロがあります'
id: why-so-many-macros

---

dbt 実行の出力には、プロジェクト内の 100 を超えるマクロが含まれます。

```shell
$ dbt run
Running with dbt=1.7.0
Found 1 model, 0 tests, 0 snapshots, 0 analyses, 138 macros, 0 operations, 0 seed files, 0 sources
```

これは、dbt が独自のプロジェクト（マクロも含む）とともに出荷されているためです。詳しくは [こちら](https://discourse.getdbt.com/t/did-you-know-dbt-ships-with-its-own-project/764) をご覧ください。
