---
title: シードをテストして文書化するにはどうすればいいですか?
description: "スキーマファイルを使用してシードをテストおよび文書化する"
sidebar_label: 'シードをテストして文書化する'
id: testing-seeds

---

シードをテストして文書化するには、[スキーマファイル](/reference/configs-and-properties)を使用し、設定を`seeds:`キーの下にネストします。

## 例

<File name='seeds/schema.yml'>

```yml
version: 2

seeds:
  - name: country_codes
    description: A mapping of two letter country codes to country names
    columns:
      - name: country_code
        tests:
          - unique
          - not_null
      - name: country_name
        tests:
          - unique
          - not_null
```

</File>
