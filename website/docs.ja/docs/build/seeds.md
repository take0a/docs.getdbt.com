---
title: "DAGにシードを追加する"
sidebar_label: "Seeds"
description: "dbt モデル用のシード データ ファイルを提供します。"
id: "seeds"
---
## 関連リファレンスドキュメント
* [シード設定](/reference/seed-configs)
* [シードプロパティ](/reference/seed-properties)
* [`seed` コマンド](/reference/commands/seed)

## 概要
シードは、dbt プロジェクト（通常は `seeds` ディレクトリ）内の CSV ファイルで、dbt は `dbt seed` コマンドを使用してこれを <Term id="data-warehouse" /> に読み込むことができます。

シードは、モデルの参照と同様に、下流のモデルから [`ref` 関数](/reference/dbt-jinja-functions/ref) を使用して参照できます。

これらの CSV ファイルは dbt リポジトリに保存されるため、バージョン管理され、コードレビューが可能です。シードは、変更頻度の低い静的データに最適です。

シードの適切な使用例:
* 国コードと国名のマッピングリスト
* 分析から除外するテストメールのリスト
* 従業員アカウント ID のリスト

dbt シードの不適切な使用例:
* CSV にエクスポートされた生データの読み込み
* 機密情報を含むあらゆる種類の本番環境データ
たとえば、個人を特定できる情報 (PII) やパスワードなどです。

## 例
dbt プロジェクトにシードファイルを読み込むには:
1. `seeds` ディレクトリに、`.csv` ファイル拡張子を付けたファイルを追加します (例: `seeds/country_codes.csv`)

<File name='seeds/country_codes.csv'>

```text
country_code,country_name
US,United States
CA,Canada
GB,United Kingdom
...
```

</File>

2. `dbt seed` [コマンド](/reference/commands/seed) を実行します。新しい <Term id="table" /> がウェアハウスのターゲットスキーマに作成され、`country_codes` という名前になります。
```
$ dbt seed

Found 2 models, 3 tests, 0 archives, 0 analyses, 53 macros, 0 operations, 1 seed file

14:46:15 | Concurrency: 1 threads (target='dev')
14:46:15 |
14:46:15 | 1 of 1 START seed file analytics.country_codes........................... [RUN]
14:46:15 | 1 of 1 OK loaded seed file analytics.country_codes....................... [INSERT 3 in 0.01s]
14:46:16 |
14:46:16 | Finished running 1 seed in 0.14s.

Completed successfully

Done. PASS=1 ERROR=0 SKIP=0 TOTAL=1
```

3. `ref` 関数を使用して下流モデルのシード値を参照します。

<File name='models/orders.sql'>

```sql
-- This refers to the table created from seeds/country_codes.csv
select * from {{ ref('country_codes') }}
```

</File>

## シードの設定
シードは `dbt_project.yml` で設定されます。利用可能な設定の完全なリストについては、[シード設定](reference/seed-configs.md) ドキュメントをご覧ください。


## シードのドキュメント化とテスト
YAMLでプロパティを宣言することで、シードをドキュメント化およびテストできます。詳細については、[シードのプロパティ](/reference/seed-properties)のドキュメントをご覧ください。

## FAQs
<FAQ path="Seeds/load-raw-data-with-seed" />
<FAQ path="Seeds/configurable-data-path" /> 
<FAQ path="Seeds/full-refresh-seed" />
<FAQ path="Tests/testing-seeds" />
<FAQ path="Seeds/seed-datatypes" />
<FAQ path="Runs/run-downstream-of-seed" />
<FAQ path="Seeds/leading-zeros-in-seed" />
<FAQ path="Seeds/build-one-seed" />
<FAQ path="Seeds/seed-hooks" />
