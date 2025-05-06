---
title: コンパイルされた SQL には多くのスペースと改行が含まれていますが、どうすれば削除できますか?
description: "空白スペースの制御"
sidebar_label: 'コンパイルされたSQLには多くの空白があります'
id: jinja-whitespace
---

これは「空白制御」と呼ばれます。

ブロックの先頭または末尾にマイナス記号 (`-`、例: `{{- ... -}}`、`{%- ... %}`、`{#- ... -#}`) を使用すると、ブロックの前後の空白が削除されます (詳細なドキュメントは [こちら](https://jinja.palletsprojects.com/page/templates/#whitespace-control) をご覧ください)。例については、[Jinja の使用に関するチュートリアル](/guides/using-jinja#use-whitespace-control-to-tidy-up-compiled-code) をご覧ください。

ご注意ください: 空白制御に関しては、よく考えも及ばないところがあります。
