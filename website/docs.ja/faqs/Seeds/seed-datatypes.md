---
title: シード内の列のデータ型を設定するにはどうすればよいですか?
description: "column_typesを使用してデータ型を設定する"
sidebar_label: 'シード内の列のデータ型を設定する'
id: seed-datatypes

---
dbt は、CSV 内のデータに基づいて各列のデータ型を推測します。

`column_types` [設定](reference/resource-configs/column_types.md) を使用して、次のように明示的にデータ型を設定することもできます:

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop: # you must include the project name
    warehouse_locations:
      +column_types:
        zipcode: varchar(5)
```

</File>
