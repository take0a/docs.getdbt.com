---
title: シードの列が変更されたため、`seed` コマンドを実行するとエラーが発生します。どうすればよいでしょうか?
description: "`--full-refresh`フラグを付けてコマンドを再実行します"
sidebar_label: 'シードの列が変更されたときのエラーをデバッグする'
id: full-refresh-seed

---
シードの列を変更した場合、`Database Error` が発生する可能性があります:

<Tabs
  defaultValue="snowflake"
  values={[
    { label: 'Snowflake', value: 'snowflake', },
    { label: 'Redshift', value: 'redshift', }
  ]
}>
<TabItem value="snowflake">

```shell
$ dbt seed
Running with dbt=1.6.0-rc2
Found 0 models, 0 tests, 0 snapshots, 0 analyses, 130 macros, 0 operations, 1 seed file, 0 sources

12:12:27 | Concurrency: 8 threads (target='dev_snowflake')
12:12:27 |
12:12:27 | 1 of 1 START seed file dbt_claire.country_codes...................... [RUN]
12:12:30 | 1 of 1 ERROR loading seed file dbt_claire.country_codes.............. [ERROR in 2.78s]
12:12:31 |
12:12:31 | Finished running 1 seed in 10.05s.

Completed with 1 error and 0 warnings:

Database Error in seed country_codes (seeds/country_codes.csv)
  000904 (42000): SQL compilation error: error line 1 at position 62
  invalid identifier 'COUNTRY_NAME'

Done. PASS=0 WARN=0 ERROR=1 SKIP=0 TOTAL=1

```

</TabItem>
<TabItem value="redshift">

```shell
$ dbt seed
Running with dbt=1.6.0-rc2
Found 0 models, 0 tests, 0 snapshots, 0 analyses, 149 macros, 0 operations, 1 seed file, 0 sources

12:14:46 | Concurrency: 1 threads (target='dev_redshift')
12:14:46 |
12:14:46 | 1 of 1 START seed file dbt_claire.country_codes...................... [RUN]
12:14:46 | 1 of 1 ERROR loading seed file dbt_claire.country_codes.............. [ERROR in 0.23s]
12:14:46 |
12:14:46 | Finished running 1 seed in 1.75s.

Completed with 1 error and 0 warnings:

Database Error in seed country_codes (seeds/country_codes.csv)
  column "country_name" of relation "country_codes" does not exist

Done. PASS=0 WARN=0 ERROR=1 SKIP=0 TOTAL=1
```

</TabItem>

</Tabs>

この場合は、次のように `--full-refresh` フラグを付けてコマンドを再実行する必要があります:

```shell
dbt seed --full-refresh
```

**なぜそうなるのでしょうか？**

通常、dbt seed を実行すると、dbt は既存の <Term id="table" /> を切り捨て、データを再挿入します。このパターンにより、下流のオブジェクト（BI ユーザーがクエリを実行している可能性のあるオブジェクト）が削除される可能性のある `drop cascade` コマンドを回避できます。

ただし、列名が変更されたり、新しい列が追加されたりすると、テーブル構造が変更されるため、これらのステートメントは失敗します。

`--full-refresh` フラグを指定すると、dbt は既存のテーブルを再構築する前に `drop cascade` を強制的に実行します。
