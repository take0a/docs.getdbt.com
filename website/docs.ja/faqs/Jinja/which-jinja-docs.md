---
title: Jinja を記述したりマクロを作成するときにはどのドキュメントを使用すればよいですか?
description: "役に立つJinjaドキュメント"
sidebar_label: '役に立つJinjaドキュメント'
id: which-jinja-docs
---

Jinja の問題で行き詰まった場合、詳細情報をどこで確認すればよいか迷うことがあります。以下のドキュメントを（順番に）確認することをお勧めします。

1. [Jinja のテンプレートデザイナーのドキュメント](https://jinja.palletsprojects.com/page/templates/): これは、使用するほとんどの Jinja に関する最適なリファレンスです。
2. [Jinja 関数リファレンス](/reference/dbt-jinja-functions): これは、dbt で Jinja に追加された追加機能について説明しています。
3. [Agate のテーブルドキュメント](https://agate.readthedocs.io/page/api/table.html): クエリの結果を操作する場合、dbt はそれを Agate テーブルとして返します。つまり、<Term id="table" /> で呼び出すメソッドは、Jinja や dbt ではなく、Agate ライブラリに属します。
