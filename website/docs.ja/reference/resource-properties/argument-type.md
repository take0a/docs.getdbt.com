---
title: type
resource_types: macro_argument
datatype: argument_type
---

import MacroArgsNote from '/snippets.ja/_validate-macro-args.md';


<File name='macros/<filename>.yml'>

```yml
version: 2

macros:
  - name: <macro name>
    arguments:
      - name: <arg name>
        type: <string>

```

</File>

## 定義

引数のデータ型。これはドキュメント作成のみを目的としており、使用できる値に制限はありません。

<MacroArgsNote />

## 例

### マクロを文書化する

<File name='macros/cents_to_dollars.sql'>

```sql
{% macro cents_to_dollars(column_name, scale=2) %}
    ({{ column_name }} / 100)::numeric(16, {{ scale }})
{% endmacro %}

```

</File>

<File name='macros/cents_to_dollars.yml'>

```yml
version: 2

macros:
  - name: cents_to_dollars
    arguments:
      - name: column_name
        type: column name or expression
        description: "The name of a column, or an expression — anything that can be `select`-ed as a column"

      - name: scale
        type: integer
        description: "The number of decimal places to round to. Default is 2."

```

</File>
