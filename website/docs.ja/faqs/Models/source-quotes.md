---
title: ソースから選択するには引用符を使用する必要があります。どうすればよいでしょうか?
description: "引用符プロパティを使用して値を引用符で囲む"
sidebar_label: '値をクォートする方法'
id: source-quotes

---

これは特にSnowflakeでよく見られます。

デフォルトでは、dbtは指定したソーステーブルのデータベース、スキーマ、または識別子を引用符で囲みません。

dbtにこれらの値を引用符で囲ませるには、[`quoting` プロパティ](/reference/resource-properties/quoting)を使用します:

<File name='models/<filename>.yml'>

```yaml
version: 2

sources:
  - name: jaffle_shop
    database: raw
    quoting:
      database: true
      schema: true
      identifier: true

    tables:
      - name: order_items
      - name: orders
        # This overrides the `jaffle_shop` quoting config
        quoting:
          identifier: false
```

</File>
