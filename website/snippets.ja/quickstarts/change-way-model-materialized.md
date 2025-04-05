dbt の最も強力な機能の 1 つは、構成値を変更するだけで、モデルがウェアハウスでマテリアライズされる方法を変更できることです。キーワードを変更することで、テーブルとビュー間で変更を加えることができます。裏でこれを行うためにデータ定義言語 (DDL) を記述する必要はありません。

デフォルトでは、すべてがビューとして作成されます。これをディレクトリ レベルでオーバーライドして、そのディレクトリ内のすべてのものを別のマテリアライズにマテリアライズすることができます。

1. `dbt_project.yml` ファイルを編集します。
    - プロジェクトの「名前」を次のように更新します:
      <File name='dbt_project.yml'>

      ```yaml
      name: 'jaffle_shop'
      ```

      </File>
    - `jaffle_shop` を設定して、その中のすべてがテーブルとしてマテリアライズされるようにします。また、`example` を設定して、その中のすべてがビューとしてマテリアライズされるようにします。`models` 設定ブロックを次のように更新します:

      <File name='dbt_project.yml'>

      ```yaml
      models:
        jaffle_shop:
          +materialized: table
          example:
            +materialized: view
      ```

      </File>
    - Click **Save**.

2. `dbt run` コマンドを入力します。これで、`customers` モデルがテーブルとして構築されるはずです！
    :::info
    これを実行するには、dbt は最初に `drop view` ステートメント (または BigQuery の API 呼び出し) を実行し、次に `create table as` ステートメントを実行する必要がありました。
    :::

3. `models/customers.sql` を編集して、`customers` モデルの `dbt_project.yml` のみを上書きし、先頭に次のスニペットを追加して、[**保存**] をクリックします:  

    <File name='models/customers.sql'>

    ```sql
    {{
      config(
        materialized='view'
      )
    }}

    with customers as (

        select
            id as customer_id
            ...

    )

    ```

    </File>

4. `dbt run` コマンドを入力します。これで、モデル `customers` がビューとして構築されるはずです。
   - BigQuery ユーザーは、マテリアライゼーションの変更を完全に適用するために、`dbt run` ではなく `dbt run --full-refresh` を実行する必要があります。
5. これをウェアハウスで有効にするには、`dbt run --full-refresh` コマンドを入力します。

### FAQs

<FAQ path="Models/available-materializations" />
<FAQ path="Project/which-materialization" />
<FAQ path="Models/available-configurations" />
