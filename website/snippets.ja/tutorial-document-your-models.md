プロジェクトに [ドキュメント](/docs/build/documentation) を追加すると、モデルを詳細に記述し、その情報をチームと共有できます。ここでは、プロジェクトに基本的なドキュメントを追加します。

1. `models/schema.yml` ファイルを更新して、以下のような説明を追加します。

    <File name='models/schema.yml'>

    ```yaml
    version: 2

    models:
      - name: customers
        description: One record per customer
        columns:
          - name: customer_id
            description: Primary key
            tests:
              - unique
              - not_null
          - name: first_order_date
            description: NULL when a customer has not yet placed an order.

      - name: stg_customers
        description: This model cleans up customer data
        columns:
          - name: customer_id
            description: Primary key
            tests:
              - unique
              - not_null

      - name: stg_orders
        description: This model cleans up order data
        columns:
          - name: order_id
            description: Primary key
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

2. `dbt docs generate` を実行して、プロジェクトのドキュメントを生成します。dbt はプロジェクトとウェアハウスをイントロスペクトして、プロジェクトに関する豊富なドキュメントを含む <Term id="json" /> ファイルを生成します。
