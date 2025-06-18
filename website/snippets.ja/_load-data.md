ここで使用するデータは、パブリック S3 バケットに CSV ファイルとして保存されています。以下の手順に従って、Snowflake アカウントでそのデータを準備し、アップロードしてください。

1. 新しい仮想ウェアハウス、2 つの新しいデータベース（1 つは生データ用、もう 1 つは将来の dbt 開発用）、2 つの新しいスキーマ（1 つは `jaffle_shop` データ用、もう 1 つは `stripe` データ用）を作成します。

    これを行うには、新しい Snowflake ワークシートのエディターに以下の SQL コマンドを入力し、UI の右上隅にある [**実行**] をクリックして実行します:
    ```sql
    create warehouse transforming;
    create database raw;
    create database analytics;
    create schema raw.jaffle_shop;
    create schema raw.stripe;
    ```

2. `raw` データベースと `jaffle_shop` および `stripe` スキーマに 3 つのテーブルを作成し、関連するデータをロードします。

    - まず、Snowflake ワークシートのエディターですべてのコンテンツ（空）を削除します。次に、次の SQL コマンドを実行して `customer` テーブルを作成します:

        ```sql 
        create table raw.jaffle_shop.customers 
        ( id integer,
          first_name varchar,
          last_name varchar
        );
        ```

    - エディター内のすべてのコンテンツを削除し、次のコマンドを実行してデータを `customer` テーブルにロードします:

        ```sql 
        copy into raw.jaffle_shop.customers (id, first_name, last_name)
        from 's3://dbt-tutorial-public/jaffle_shop_customers.csv'
        file_format = (
            type = 'CSV'
            field_delimiter = ','
            skip_header = 1
            ); 
        ```
    - エディター内のすべてのコンテンツを削除し (空)、次のコマンドを実行して `orders` テーブルを作成します:
        ```sql
        create table raw.jaffle_shop.orders
        ( id integer,
          user_id integer,
          order_date date,
          status varchar,
          _etl_loaded_at timestamp default current_timestamp
        );
        ```

    - エディター内のすべてのコンテンツを削除し、次のコマンドを実行してデータを `orders` テーブルにロードします:
        ```sql
        copy into raw.jaffle_shop.orders (id, user_id, order_date, status)
        from 's3://dbt-tutorial-public/jaffle_shop_orders.csv'
        file_format = (
            type = 'CSV'
            field_delimiter = ','
            skip_header = 1
            );
        ```
    - エディター内のすべてのコンテンツを削除し (空)、次のコマンドを実行して `payment` テーブルを作成します:
        ```sql
        create table raw.stripe.payment 
        ( id integer,
          orderid integer,
          paymentmethod varchar,
          status varchar,
          amount integer,
          created date,
          _batched_at timestamp default current_timestamp
        );
        ```
    - エディター内のすべてのコンテンツを削除し、次のコマンドを実行してデータを `payment` テーブルにロードします:
        ```sql
        copy into raw.stripe.payment (id, orderid, paymentmethod, status, amount, created)
        from 's3://dbt-tutorial-public/stripe_payments.csv'
        file_format = (
            type = 'CSV'
            field_delimiter = ','
            skip_header = 1
            );
        ```
3. 以下のSQLクエリを実行して、データがロードされていることを確認します。それぞれの出力が表示されることを確認してください。
    ```sql
    select * from raw.jaffle_shop.customers;
    select * from raw.jaffle_shop.orders;
    select * from raw.stripe.payment;   
    ```