---
resource_types: [tests]
id: "store_failures_as"
---

`test` リソースタイプの場合、`store_failures_as` は、テストの失敗をデータベースに保存する方法を指定するオプションの設定です。[`store_failures`](/reference/resource-configs/store_failures) も設定されている場合、`store_failures_as` が優先されます。

サポートされている 3 つの値は次のとおりです。

- `ephemeral` &mdash; データベースに何も保存しません (デフォルト)
- `table` &mdash; テストの失敗をデータベース テーブルとして保存します
- `view` &mdash; テストの失敗をデータベース ビューとして保存します

`store_failures` と同様に、単独のテスト (.sql ファイル)、汎用テスト (.yml ファイル)、dbt_project.yml など、すべての場所で設定できます。

### 例

#### Singular テスト

`tests/singular/check_something.sql` ファイル内の [Singular テスト](https://docs.getdbt.com/docs/build/data-tests#singular-data-tests)

```sql
{{ config(store_failures_as="table") }}

-- custom singular test
select 1 as id
where 1=0
```

#### 汎用テスト

[汎用テスト](https://docs.getdbt.com/docs/build/data-tests#generic-data-tests) (`models/_models.yml` ファイル内)

```yaml
models:
  - name: my_model
    columns:
      - name: id
        tests:
          - not_null:
              config:
                store_failures_as: view
          - unique:
              config:
                store_failures_as: ephemeral
```

#### プロジェクトレベル

`dbt_project.yml` の設定

```yaml
name: "my_project"
version: "1.0.0"
config-version: 2
profile: "sandcastle"

tests:
  my_project:
    +store_failures_as: table
    my_subfolder_1:
      +store_failures_as: view
    my_subfolder_2:
      +store_failures_as: ephemeral
```

### "上書き設定

他のほとんどの設定と同様に、`store_failures_as` は階層的に適用された場合、「上書き」されます。より具体的な値が利用可能な場合は、より具体的な値でない値が完全に置き換えられます。

追加リソース:

- [データテスト設定](/reference/data-test-configs#related-documentation)
- [データテスト固有の設定](/reference/data-test-configs#test-data-specific-configurations)
- [dbt_project.yml でのモデルディレクトリの設定](/reference/model-configs#configuring-directories-of-models-in-dbt_projectyml)
- [設定の継承](/reference/define-configs#config-inheritance)
