---
resource_types: [models]
datatype: access
---

<File name='models/<schema>.yml'>

```yml
version: 2

models:
  - name: model_name
    access: private | protected | public
```

</File>

アクセス修飾子は、`dbt_project.yml` などの設定ファイル、または `properties.yml` でモデルごとに個別に適用できます。サブフォルダにアクセス設定を適用すると、そのサブフォルダ内のすべてのモデルのデフォルトが変更されるため、この動作を意図していることを確認してください。個々のモデルへのアクセスを設定する場合、グループまたはサブフォルダにはさまざまなアクセスレベルが含まれる可能性があるため、モデルに `access: public` を指定する場合は、この動作を意図していることを確認してください。

アクセスを設定するには、複数の方法があります。

- `properties.yml` で、従来の方法を使用する場合：

  <File name='models/properties_my_public_model.yml'>
  
  ```yml
  version: 2
  
  models:
    - name: my_public_model
      access: public # Older method, still supported
      
  ```
  </File>
  
- `properties.yml` で新しいメソッドを使用します（バージョン 1.7 以降）。同じモデルに対して、古いメソッドと新しいメソッドのどちらか一方のみを使用してください。

  <File name='models/properties_my_public_model.yml'>
  
  ```yml
  version: 2
  
  models:
    - name: my_public_model
      config:
        access: public # newly supported in v1.7
      
  ```
  </File>


- `dbt_project.yml` 内:

  <File name='dbt_project.yml'>
  
  ```yml
  models:
    my_project_name:
      subfolder_name:
        +group: my_group
        +access: private  # sets default for all models in this subfolder
  ```
  </File>

- `my_public_model.sql` ファイル内:

  <File name='models/my_public_model.sql'>
  
  ```sql
  -- models/my_public_model.sql
  
  {{ config(access = "public") }}
  
  select ...
  ```
  </File>

`access` を定義した後、本番ジョブを再実行して変更を適用します。

## 定義

プロパティを宣言するモデルのアクセスレベル。

一部のモデル（すべてではありません）は、[ref](/reference/dbt-jinja-functions/ref) 関数を介して [groups](/docs/build/groups) に参照されるように設計されています。

| Access    | Referenceable by              |
|-----------|-------------------------------|
| private   | 同じグループ                    |
| protected | 同じプロジェクト/パッケージ         |
| public    | 任意のグループ、パッケージ、またはプロジェクト。定義したら、本番ジョブを再実行して変更を適用します。 |

サポートされているアクセス範囲外でモデルを参照しようとすると、エラーが表示されます:

```shell
dbt run -s marketing_model
...
dbt.exceptions.DbtReferenceError: Parsing Error
  Node model.jaffle_shop.marketing_model attempted to reference node model.jaffle_shop.finance_model, 
  which is not allowed because the referenced node is private to the finance group.
```

## デフォルト

デフォルトでは、すべてのモデルは「protected」です。つまり、同じプロジェクト内の他のモデルから参照できます。

## 関連ドキュメント

* [モデルアクセス](/docs/collaborate/govern/model-access#groups)
* [グループ設定](/reference/resource-configs/group)
