---
title: "DAGにグループを追加する"
sidebar_label: "Groups"
id: "groups"
description: "dbt プロジェクトでグループを定義すると、暗黙的な関係が明示的なグループ化に変換されます。"
keywords:
  - groups access mesh
---

グループは、dbt DAG 内のノードの集合です。
グループには名前が付けられ、各グループには「オーナー」が存在します。
グループは、[プライベート](/reference/resource-configs/access)モデルへのアクセスを制限することで、チーム内およびチーム間での意図的なコラボレーションを可能にします。

グループのメンバーには、モデル、テスト、シード、スナップショット、分析、メトリクスなどが含まれます。
(ソースとエクスポージャーは含まれません。)
各ノードは1つのグループにのみ所属できます。

### グループの宣言

グループは `.yml` ファイルで定義され、`groups:` キーの下にネストされます。

<File name='models/marts/finance/finance.yml'>

```yaml
groups:
  - name: finance
    owner:
      # 'name' or 'email' is required; additional properties allowed
      email: finance@jaffleshop.com
      slack: finance-data
      github: finance-data-team
```

</File>

### グループへのモデルの追加

`group` 設定を使用して、1 つ以上のモデルをグループに追加します。

<Tabs>
<TabItem value="project" label="Project-level">

<File name='dbt_project.yml'>

```yml
models:
  marts:
    finance:
      +group: finance
```

</File>

</TabItem>

<TabItem value="model-yaml" label="Model-level">

<File name='models/schema.yml'>

```yml
models:
  - name: model_name
    config:
      group: finance
```

</File>

</TabItem>

<TabItem value="model-file" label="In-file">

<File name='models/model_name.sql'>

```sql
{{ config(group = 'finance') }}

select ...
```

</File>

</TabItem>

</Tabs>

### グループ内のモデルの参照

デフォルトでは、グループ内のすべてのモデルには `protected` [アクセス修飾子](/reference/resource-configs/access) が付与されます。
つまり、同じプロジェクト内の任意のグループに属する下流リソースから、[`ref`](/reference/dbt-jinja-functions/ref) 関数を使用して参照できます。
グループ化されたモデルの `access` プロパティが `private` に設定されている場合、そのグループ内のリソースのみがそのモデルを参照できます。

<File name='models/schema.yml'>

```yml
models:
  - name: finance_private_model
    access: private
    config:
      group: finance

  # in a different group!
  - name: marketing_model
    config:
      group: marketing
```
</File>

<File name='models/marketing_model.sql'>

```sql
select * from {{ ref('finance_private_model') }}
```
</File>

```shell
$ dbt run -s marketing_model
...
dbt.exceptions.DbtReferenceError: Parsing Error
  Node model.jaffle_shop.marketing_model attempted to reference node model.jaffle_shop.finance_private_model, 
  which is not allowed because the referenced node is private to the finance group.
```

## 関連ドキュメント

* [モデルアクセス](/docs/collaborate/govern/model-access#groups)
* [グループ設定](/reference/resource-configs/group)
* [グループ選択](/reference/node-selection/methods#group)
