---
title: シードファイルを追加する
id: add-a-seed
description: プロジェクトにシードファイルを追加する方法を学ぶ
---

1. シードファイルを追加します:

<File name='seeds/country_codes.csv'>

```text
country_code,country_name
US,United States
CA,Canada
GB,United Kingdom
...
```

</File>

2. `dbt seed` を実行する
3. 下流モデルでモデルを参照する

<File name='models/something.sql'>

```sql
select * from {{ ref('country_codes') }}
```

</File>
