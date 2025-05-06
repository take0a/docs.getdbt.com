---
title: dbt で使用できるテストは何ですか?    
description: "dbtで使用するテストの種類"
sidebar_label: 'dbtで使用できるテスト'
id: available-tests

---
dbt には、以下のテストが標準で付属しています。

* `unique`
* `not_null`
* `accepted_values`
* `relationships` (参照整合性など)

独自の [カスタムスキーマデータテスト](/docs/build/data-tests) を作成することもできます。

[dbt-utils パッケージ](https://github.com/dbt-labs/dbt-utils?#generic-tests) には、追加のカスタムスキーマテストがオープンソース化されています。これらのテストをプロジェクトで利用できるようにする方法については、[パッケージ](/docs/build/packages) のドキュメントをご覧ください。

現時点ではデータテストをドキュメント化することはできませんが、dbt コミュニティがアイデアを共有している [dbt Core のディスカッション](https://github.com/dbt-labs/dbt-core/issues/2578) を確認することをお勧めします。
