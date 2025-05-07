---
datatype: string
description: "dbt のプロファイル構成を理解するには、このガイドをお読みください。"
---
<File name='dbt_project.yml'>

```yml
profile: string
```

</File>

## 定義
dbt プロジェクトが <Term id="data-warehouse" /> に接続するために使用するプロファイル。
* dbt Cloud で開発している場合: この構成は適用されません。
* ローカルで開発している場合: コマンドラインオプション (`--profile` など) が指定されていない限り、この構成は必須です。

## 関連ガイド
* [コマンドラインを使用したウェアハウスへの接続](/docs/core/connect-data-platform/connection-profiles#connecting-to-your-warehouse-using-the-command-line)

## 推奨事項
多くの場合、組織には <Term id="data-warehouse" /> が 1 つしかありません。そのため、プロファイル名には組織名を `snake_case` で表記するのが賢明です。例:
* `profile: acme`
* `profile: jaffle_shop`

特に複数のデータウェアハウスを所有している場合は、プロフィール名にデータウェアハウス技術の名称を含めることをお勧めします。例：
* `profile: acme_snowflake`
* `profile: jaffle_shop_bigquery`
* `profile: jaffle_shop_redshift`
