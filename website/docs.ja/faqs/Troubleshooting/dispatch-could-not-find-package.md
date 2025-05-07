---
title: "[Error] Could not find my_project package"
description: "パッケージからマクロが欠落しています"
sidebar_label: 'パッケージエラーが見つかりません'
id: dispatch-could-not-find-package

---

プロジェクトレベルの `dispatch` 設定の `search_order` にパッケージ名が含まれている場合、dbt はそのパッケージにディスパッチ可能なマクロが含まれていると想定します。含まれているパッケージにマクロがまったく含まれていない場合、dbt は次のようなエラーを発生させます:

```shell
Compilation Error
  In dispatch: Could not find package 'my_project'
```

これはパッケージまたはルートプロジェクトが欠落しているという意味ではなく、その中のマクロが欠落しているため、`dispatch` で利用可能な検索スペースから欠落しているという意味です。

上記の手順を試してもこの現象が続く場合は、サポートチーム（support@getdbt.com）までお問い合わせください。喜んでお手伝いいたします。
