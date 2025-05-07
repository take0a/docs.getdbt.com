---
datatype: string
description: "dbt での name の構成を理解するには、このガイドをお読みください。"
required: True
---

<File name='dbt_project.yml'>

```yml
name: string
```

</File>

## 定義
**必須設定**

dbt プロジェクトの名前。文字、数字、アンダースコアのみで構成でき、数字で始まる名前は指定できません。

## 推奨事項
多くの場合、組織には dbt プロジェクトが 1 つだけ存在します。そのため、プロジェクト名は組織名を `snake_case` で表記するのが賢明です。例:
* `name: acme`
* `name: jaffle_shop`
* `name: evilcorp`


## トラブルシューティング
### 無効なプロジェクト名

```
Encountered an error while reading the project:
  ERROR: Runtime Error
  at path ['name']: 'jaffle-shop' does not match '^[^\\d\\W]\\w*$'
Runtime Error
  Could not run dbt
```

このプロジェクトには次のものが含まれます:

<File name='dbt_project.yml'>

```yml
name: jaffle-shop
```

</File>

この場合は、プロジェクト名を `snake_case` に変更します。

<File name='dbt_project.yml'>

```yml
name: jaffle_shop
```

</File>
