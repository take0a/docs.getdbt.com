---
title: ターゲット スキーマ以外のスキーマでシードを構築したり、シードを複数のスキーマに分割したりできますか?
description: "dbt_project.yml ファイルでスキーマ設定を使用する"
sidebar_label: 'ターゲット スキーマ外のスキーマでシードを構築する'
id: seed-custom-schemas

---

はい！`dbt_project.yml` ファイルで [schema](reference/resource-configs/schema.md) 構成を使用してください。

<File name='dbt_project.yml'>

```yml

name: jaffle_shop
...

seeds:
  jaffle_shop:
    schema: mappings # all seeds in this project will use the schema "mappings" by default
    marketing:
      schema: marketing # seeds in the "seeds/marketing/" subdirectory will use the schema "marketing"
```

</File>
