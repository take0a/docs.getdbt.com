---
title: マクロを文書化するにはどうすればいいですか?
description: "マクロを文書化するためにスキーマファイルを使用できます"
sidebar_label: 'マクロを文書化する'
id: documenting-macros
---

import MacroArgsNote from '/snippets/_validate-macro-args.md';

マクロを文書化するには、[スキーマファイル](/reference/macro-properties)を使用し、`macros:`キーの下に構成をネストします。

## Example

<File name='macros/schema.yml'>

```yml
version: 2

macros:
  - name: cents_to_dollars
    description: A macro to convert cents to dollars
    arguments:
      - name: column_name
        type: string
        description: The name of the column you want to convert
      - name: precision
        type: integer
        description: Number of decimal places. Defaults to 2.
```

</File>

<MacroArgsNote />

## カスタムマテリアライゼーションをドキュメント化する

[カスタムマテリアライゼーション](/guides/create-new-materializations)を作成すると、dbt は次の形式の関連マクロを作成します:

```
materialization_{materialization_name}_{adapter}
```

カスタム マテリアライゼーションを文書化するには、前述の形式を使用して、文書化する関連マクロ名を決定します。

<File name='macros/properties.yml'>

```yaml
version: 2

macros:
  - name: materialization_my_materialization_name_default
    description: A custom materialization to insert records into an append-only table and track when they were added.
  - name: materialization_my_materialization_name_xyz
    description: A custom materialization to insert records into an append-only table and track when they were added.
```

</File>
