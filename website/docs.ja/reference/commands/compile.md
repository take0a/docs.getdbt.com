---
title: "dbt compile コマンドについて"
description: "dbt compile コマンドは、モデル、テスト、および分析ファイルから実行可能な SQL を作成します。"
sidebar_label: "compile"
id: "compile"
---

`dbt compile` は、ソースファイル `model`、`test`、`analysis` から実行可能な SQL を生成します。これらのコンパイル済み SQL ファイルは、dbt プロジェクトの `target/` ディレクトリにあります。

`compile` コマンドは、次の場合に便利です。

1. モデルファイルのコンパイル済み出力を視覚的に検査する。これは、複雑な Jinja ロジックやマクロの使用を検証するのに役立ちます。
2. コンパイル済み SQL を手動で実行する。モデルまたはスキーマテストのデバッグ中に、バグの原因を特定するために、基礎となる `select` ステートメントを実行することが役立つことがよくあります。
3. `analysis` ファイルをコンパイルする。分析ファイルの詳細については、[こちら](/docs/build/analyses) を参照してください。

よくある誤解：
- `dbt compile` は、`dbt run` やその他のビルドコマンドの前提条件ではありません。これらのコマンドは、コンパイル処理を自動で行います。
- データ ウェアハウスに接続せずに、dbt でプロジェクト コードを読み取って検証するだけの場合は、代わりに `dbt parse` を使用します。

### 対話型コンパイル

dbt v1.5 以降では、CLI で `compile` を「対話型」に実行できるようになりました。これにより、ノードまたは任意の dbt-SQL クエリのコンパイル済みコードを表示できます。
- `--select` で特定のノードを名前で選択
- `--inline` で任意の dbt-SQL クエリを選択

これにより、コンパイルされた SQL が `target/` ディレクトリに書き込まれるだけでなく、ターミナルにもログ出力されます。

例:

```bash
dbt compile --select "stg_orders"                           
dbt compile --inline "select * from {{ ref('raw_orders') }}"
```

次を返します:

```bash
dbt compile --select "stg_orders"                           

21:17:09  Running with dbt=1.7.5
21:17:09  Registered adapter: postgres=1.7.5
21:17:09  Found 5 models, 3 seeds, 20 tests, 0 sources, 0 exposures, 0 metrics, 401 macros, 0 groups, 0 semantic models
21:17:09  
21:17:09 Concurrency: 24 threads (target='dev')
21:17:09  
21:17:09  Compiled node 'stg_orders' is:
with source as (
    select * from "jaffle_shop"."main"."raw_orders"

),

renamed as (

    select
        id as order_id,
        user_id as customer_id,
        order_date,
        status

    from source

)

select * from renamed
```

```bash
dbt compile --inline "select * from {{ ref('raw_orders') }}"

18:15:49  Running with dbt=1.7.5
18:15:50  Registered adapter: postgres=1.7.5
18:15:50  Found 5 models, 3 seeds, 20 tests, 0 sources, 0 exposures, 0 metrics, 401 macros, 0 groups, 0 semantic models
18:15:50  
18:15:50  Concurrency: 5 threads (target='postgres')
18:15:50  
18:15:50  Compiled inline node is:
select * from "jaffle_shop"."main"."raw_orders"
```

このコマンドは、データプラットフォームにアクセスしてキャッシュ関連のメタデータを取得し、イントロスペクティブクエリを実行します。以下のフラグを使用します。
- `--no-populate-cache` フラグを使用すると、初期キャッシュポピュレーションが無効になります。メタデータが必要な場合はキャッシュミスとなり、dbt はメタデータクエリを実行する必要があります。これは `dbt` フラグなので、プレフィックスとして `dbt` を追加する必要があります。例: `dbt --no-populate-cache`
- `--no-introspect` フラグを使用すると、[イントロスペクティブクエリ](/faqs/Warehouse/db-connection-dbt-compile#introspective-queries) が無効になります。モデルの定義でイントロスペクティブクエリの実行が必要な場合、dbt はエラーを生成します。これは `dbt compile` フラグなので、プレフィックスとして `dbt compile` を追加する必要があります。例: `dbt compile --no-introspect`


### FAQs
<FAQ path="Warehouse/db-connection-dbt-compile" />
