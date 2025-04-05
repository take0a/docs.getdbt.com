プロジェクトに [テスト](/docs/build/data-tests) を追加すると、モデルが正しく動作していることを検証できます。

プロジェクトにテストを追加するには:

1. `models` ディレクトリに `models/schema.yml` という名前の新しい YAML ファイルを作成します。
2. ファイルに次の内容を追加します:

    <File name='models/schema.yml'>

    ```yaml
    version: 2

    models:
      - name: customers
        columns:
          - name: customer_id
            tests:
              - unique
              - not_null

      - name: stg_customers
        columns:
          - name: customer_id
            tests:
              - unique
              - not_null

      - name: stg_orders
        columns:
          - name: order_id
            tests:
              - unique
              - not_null
          - name: status
            tests:
              - accepted_values:
                  values: ['placed', 'shipped', 'completed', 'return_pending', 'returned']
          - name: customer_id
            tests:
              - not_null
              - relationships:
                  to: ref('stg_customers')
                  field: customer_id

    ```

    </File>

3. `dbt test` を実行し、すべてのテストが成功したことを確認します。

`dbt test` を実行すると、dbt は YAML ファイルを反復処理し、各テストのクエリを構築します。各クエリは、テストに失敗したレコードの数を返します。この数が 0 の場合、テストは成功です。

#### FAQs

<FAQ path="Tests/available-tests" alt_header="What tests are available for me to use in dbt? Can I add my own custom tests?" />
<FAQ path="Tests/test-one-model" />
<FAQ path="Runs/failed-tests" />
<FAQ path="Project/schema-yml-name" alt_header="Does my test file need to be named `schema.yml`?" />
<FAQ path="Project/why-version-2" />
<FAQ path="Tests/recommended-tests" />
<FAQ path="Tests/when-to-test" />
