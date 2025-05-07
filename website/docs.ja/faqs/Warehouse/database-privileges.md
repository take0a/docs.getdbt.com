---
title: dbt を使用するには、データベース ユーザーにどのような権限が必要ですか?
description: "dbt を使用するためのデータベース権限"
sidebar_label: 'dbt を使用するためのデータベース権限'
id: database-privileges

---
ユーザーには以下の権限が必要です。
* ウェアハウス内の生データ（つまり、変換対象のデータ）から `select` する
* スキーマを `create` し、そのスキーマ内にテーブル/ビューを作成する¹
* システム <Term id="view">views</Term> を読み取ってドキュメントを生成する（つまり、`information_schema` 内のビュー）。

Postgres、Redshift、Databricks、Snowflake では、一連の `grants` を使用して、ユーザーに適切な権限が付与されていることを確認します。これらのウェアハウスの [権限の例](/reference/database-permissions/about-database-permissions) をご確認ください。

BigQuery では、「BigQuery ユーザー」ロールを使用してこれらの権限を割り当てます。

---
¹あるいは、別のユーザーがdbtユーザー用のスキーマを作成し、そのスキーマ内で作成する権限をユーザーに付与することもできます。実装が比較的簡単なため、通常はdbtユーザーにスキーマ作成権限を付与することをお勧めします。
