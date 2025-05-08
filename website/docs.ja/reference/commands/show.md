---
title: "dbt show コマンドについて"
sidebar_label: "show"
id: "show"
---

`dbt show` を使用すると、次の操作を実行できます:
- `model`、`test`、`analysis`、または `--inline` で渡された任意の dbt-SQL クエリの dbt-SQL 定義をコンパイルする
- `dbt show` は [Python (dbt-py)](/docs/build/python-models) モデルをサポートしていません。
- データウェアハウスに対してクエリを実行する
- ターミナルで結果をプレビューする

デフォルトでは、`dbt show` はクエリ結果の最初の 5 行を表示します。これは、フラグ `--limit n` を渡すことでカスタマイズできます。`n` は表示する行数です。

プレビュークエリの結果は、データウェアハウスにマテリアライズされず、dbt ファイルにも保存されません。dbt のログにのみ含まれ、ターミナルに表示されます。また、モデルをプレビューする場合、dbt は常にソースからコンパイルされたクエリをコンパイルして実行することに注意してください。モデルを実行した直後であっても、既にマテリアライズされたデータベースリレーションからは選択されません。(将来的にはサポートされる可能性があります。ご興味があれば、[dbt-core#7391](https://github.com/dbt-labs/dbt-core/issues/7391) に賛成票を投じるか、コメントを投稿してください。)

例:

```
dbt show --select "model_name.sql"
```

または

```
dbt show --inline "select * from {{ ref('model_name') }}"
```

以下は、`stg_orders` という名前のモデルの `dbt show` 出力の例です:

```bash
dbt show --select "stg_orders"
21:17:38 Running with dbt=1.5.0-b5
21:17:38 Found 5 models, 20 tests, 0 snapshots, 0 analyses, 425 macros, 0 operations, 3 seed files, 0 sources, 0 exposures, 0 metrics, 0 groups
21:17:38
21:17:38 Concurrency: 24 threads (target='dev')
21:17:38
21:17:38 Previewing node 'stg_orders' :
| order_id | customer_id | order_date | status    |
|----------+-------------+------------+--------   |
| 1        |           1 | 2023-01-01 | returned  |
| 2        |           3 | 2023-01-02 | completed |
| 3        |          94 | 2023-01-03 | completed |
| 4        |          50 | 2023-01-04 | completed |
| 5        |          64 | 2023-01-05 | completed |

```

たとえば、失敗したテストがあるモデルを構築したばかりの場合は、ターミナル内でテストの失敗をすぐにプレビューして、重複している `id` の値を見つけることができます:

```bash
$ dbt build -s "my_model_with_duplicates"
13:22:47 .0
...
13:22:48 Completed with 1 error and 0 warnings:
13:22:48
13:22:48 Failure in test unique_my_model_with_duplicates (models/schema.yml)
13:22:48   Got 1 result, configured to fail if not 0
13:22:48
13:22:48   compiled code at target/compiled/my_dbt_project/models/schema.yml/unique_my_model_with_duplicates_id.sql
13:22:48
13:22:48 Done. PASS=1 WARN=0 ERROR=1 SKIP=0 TOTAL=2

$ dbt show -s "unique_my_model_with_duplicates_id"
13:22:53 Running with dbt=1.5.0
13:22:53 Found 4 models, 2 tests, 0 snapshots, 0 analyses, 309 macros, 0 operations, 0 seed files, 0 sources, 0 exposures, 0 metrics, 0 groups
13:22:53
13:22:53 Concurrency: 5 threads (target='dev')
13:22:53
13:22:53 Previewing node 'unique_my_model_with_duplicates_id':
| unique_field | n_records |
| ------------ | --------- |
|            1 |         2 |

```
