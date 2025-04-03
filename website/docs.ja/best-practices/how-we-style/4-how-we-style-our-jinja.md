---
title: Jinja のスタイル
id: 4-how-we-style-our-jinja
---

## Jinja スタイルガイド

- 🫧 Jinja 区切り文字を使用する場合は、区切り文字の内側にスペースを入れます。たとえば、`{{this}}` ではなく `{{ this }}` を使用します。
- 🆕 改行を使用して、Jinja の論理ブロックを視覚的に示すことができます。
- 4️⃣ Jinja ブロック内に 4 つのスペースをインデントして、内部のコードがそのブロックで囲まれていることを視覚的に示します。
- ❌ Jinja の空白制御について (あまり) 心配せず、プロジェクト コードが読みやすいことに集中してください。空白制御を気にしないことで節約できる時間は、完璧ではない可能性のあるコンパイル済みコードに費やす時間よりもはるかに重要です。

## Jinja のスタイルの例

```jinja
{% macro make_cool(uncool_id) %}

    do_cool_thing({{ uncool_id }})

{% endmacro %}
```

```sql
select
    entity_id,
    entity_type,
    {% if this %}

        {{ that }},

    {% else %}

        {{ the_other_thing }},

    {% endif %}
    {{ make_cool('uncool_id') }} as cool_id
```
