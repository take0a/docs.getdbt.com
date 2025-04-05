---
title: "DuckDBのセットアップ"
description: "Read this guide to learn about the DuckDB warehouse setup in dbt."
meta:
  maintained_by: Community
  authors: 'Josh Wills (https://github.com/jwills)'
  github_repo: 'duckdb/dbt-duckdb'
  pypi_package: 'dbt-duckdb'
  min_core_version: 'v1.0.1'
  cloud_support: Not Supported
  min_supported_version: 'DuckDB 0.3.2'
  slack_channel_name: '#db-duckdb'
  slack_channel_link: 'https://getdbt.slack.com/archives/C039D1J1LA2'
  platform_name: 'Duck DB'
  config_page: '/reference/resource-configs/no-configs'
---

:::info コミュニティプラグイン

一部のコア機能は制限される可能性があります。貢献に興味がある場合は、以下にリストされている各リポジトリのソース コードを確認してください。

:::

import SetUpPages from '/snippets.ja/_setup-pages-intro.md';

<SetUpPages meta={frontMatter.meta} />


## dbt-duckdb を使用して DuckDB に接続する

[DuckDB](http://duckdb.org) は SQLite に似た組み込みデータベースですが、OLTP ではなく OLAP スタイルの分析用に設計されています。プロファイルで必要な構成パラメータは (`type: duckdb` に加えて) `path` フィールドのみです。これは、DuckDB データベース ファイル (および関連する先行書き込みログ) を書き込むローカル ファイル システム上のパスを参照する必要があります。デフォルト (`main` と呼ばれます) 以外のスキーマを使用する場合は、`schema` パラメータを指定することもできます。

`DuckDBCredentials` クラスには、親の `Credentials` クラスとの一貫性を保つために `database` フィールドも定義されていますが、デフォルトは `main` であり、別の値に設定すると完全に予測できない異常な事態が発生する可能性があるため、変更は避けてください。

バージョン 1.2.3 以降では、プロファイルの `extensions` フィールドにリストすることで、サポートされている [DuckDB 拡張機能](https://duckdb.org/docs/extensions/overview) をすべてロードできます。また、ロードされた拡張機能でサポートされているオプションを含む、追加の [DuckDB 構成オプション](https://duckdb.org/docs/sql/configuration) を `settings` フィールドで設定することもできます。

たとえば、AWS アクセス キーとシークレットを使用して `s3` に接続し、`parquet` ファイルの読み取り/書き込みを行うには、プロファイルは次のようになります:

<File name='profiles.yml'>

```yaml
your_profile_name:
  target: dev
  outputs:
    dev:
      type: duckdb
      path: 'file_path/database_name.duckdb'
      extensions:
        - httpfs
        - parquet
      settings:
        s3_region: my-aws-region
        s3_access_key_id: "{{ env_var('S3_ACCESS_KEY_ID') }}"
        s3_secret_access_key: "{{ env_var('S3_SECRET_ACCESS_KEY') }}"
```

</File>


