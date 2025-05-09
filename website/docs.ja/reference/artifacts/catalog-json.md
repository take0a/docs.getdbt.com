---
title: "カタログJSONファイル"
sidebar_label: "Catalog"
---

**現在のスキーマ**: [`v1`](https://schemas.getdbt.com/dbt/catalog/v1.json)

**生成元:** [`docs generate`](/reference/commands/cmd-docs)

このファイルには、プロジェクト内のリソースによって生成および定義されたテーブルと<Term id="view">ビュー</Term>に関する、<Term id="data-warehouse" /> からの情報が含まれています。現在、dbt はこのファイルを使用して、[ドキュメントサイト](/docs/collaborate/build-and-view-your-docs) に列タイプや<Term id="table" /> 統計などのメタデータを入力します。

### 最上位キー

- [`metadata`](/reference/artifacts/dbt-artifacts#common-metadata)
- `nodes`: dbt モデル、シード、スナップショットに対応するデータベースオブジェクトに関する情報を含む辞書。
- `sources`: dbt ソースに対応するデータベースオブジェクトに関する情報を含む辞書。
- `errors`: `dbt docs generate` 実行中にメタデータクエリを実行中に発生したエラー。

### リソースの詳細

`sources` と `nodes` 内の各辞書キーは、リソースの `unique_id` です。ネストされた各リソースには、以下の要素が含まれます。
- `unique_id`: `<resource_type>.<package>.<resource_name>`。辞書キーと同じで、[マニフェスト](/reference/artifacts/manifest-json) 内の `nodes` と `sources` にマッピングされます。
- `metadata`
    - `type`: テーブル、ビューなど。
    - `database`
    - `schema`
    - `name`
    - `comment`
    - `owner`
- `columns` (配列)
    - `name`
    - `type`: データ型
    - `comment`
    - `index`: 序数
- `stats`: データベースとリレーションの種類によって異なります。
