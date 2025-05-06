---
title: Jinja で列名を引用符で囲む必要があるのはなぜですか?
description: "文字列を渡すには引用符を使用してください"
sidebar_label: 'Jinjaで列名を引用符で囲む理由'
id: quoting-column-names
---

[マクロの例](/docs/build/jinja-macros#macros)では、列名 `amount` を引用符で囲んで渡しました。

```sql
{{ cents_to_dollars('amount') }} as amount_usd
```

_文字列_ `'amount'` をマクロに渡すには引用符を使用する必要があります。

引用符がない場合、Jinjaパーサーは `amount` という変数を探します。しかし、この変数は存在しないため、コンパイル時に何も返されません。

Jinjaでの引用符の使い方には、慣れるまで少し時間がかかるかもしれません。ルールとして、Jinjaの式または文（つまり `{% ... %}` または `{{ ... }}` 内）では、文字列の引数には引用符を使用する必要があります。

Jinjaでは、シングルクォーテーションとダブルクォーテーションは同じ意味です。適切に対応させてください。

また、変数を引数として渡す必要がある場合は、[中括弧をネストしないでください](/best-practices/dont-nest-your-curlies)。