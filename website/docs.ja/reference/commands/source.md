---
title: "dbt source コマンドについて"
sidebar_label: "source"
id: "source"
---

`dbt source` コマンドは、ソースデータの操作に役立つサブコマンドを提供します。このコマンドには、`dbt source freshness` というサブコマンドが1つあります。



### dbt source freshness

dbt プロジェクトが [ソースを使用して構成](/docs/build/sources) されている場合、`dbt source freshness` コマンドは定義済みのすべてのソーステーブルをクエリし、これらのテーブルの「鮮度」を判断します。テーブルが古い場合（ソースに指定された `freshness` 構成に基づきます）、dbt はそれに応じて警告またはエラーを報告します。ソース <Term id="table" /> が古い状態の場合、dbt はゼロ以外の終了コードで終了します。

[ソース鮮度コマンド](/reference/commands/source#source-freshness-commands) を使用して、取得するデータが古くなったり期限切れになったりしていないことを確認することもできます。

### ソースの鮮度を設定する

以下の例は、dbt でソースの鮮度を設定する方法を示しています。詳細については、[ソースの鮮度を宣言する](/docs/build/sources#declaring-source-freshness) を参照してください。

<File name='models/<filename>.yml'>

```yaml

version: 2

sources:
  - name: jaffle_shop
    database: raw

    freshness:
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}

    loaded_at_field: _etl_loaded_at

    tables:
      - name: customers

      - name: orders
        freshness:
          warn_after: {count: 6, period: hour}
          error_after: {count: 12, period: hour}
          filter: datediff('day', _etl_loaded_at, current_timestamp) < 2

      - name: product_skus
        freshness: null

```
</File>

これは、データパイプラインの健全性をモニタリングするのに役立ちます。

dbt Cloud ジョブの [**設定**] ページの [**実行設定**] セクションでソースの鮮度を設定することもできます。詳細については、[ソース鮮度スナップショットの有効化](/docs/deploy/source-freshness#enabling-source-freshness-snapshots) をご覧ください。

### Source freshness コマンド

Source freshness コマンドを使用すると、最新かつ関連性が高く、正確な情報を確実に受け取ることができます。

使用できる代表的なコマンドは以下のとおりです:

| **Command**                                                                 | **Description**                  | 
| ----------------------------------------------------------------------------| ---------------------------------|
|[`dbt source freshness`](/reference/commands/source#dbt-source-freshness)    | すべてのソースの「鮮度」をチェックします。 |
|[`dbt source freshness --output target/source_freshness.json`](/reference/commands/source#configuring-source-freshness-output)|「鮮度」情報を別のパスに出力します。|
|[`dbt source freshness --select "source:source_name"`](/reference/commands/source#specifying-sources-to-snapshot)|特定のソースの「鮮度」をチェックします。|

### スナップショットを作成するソースの指定

デフォルトでは、`dbt source freshness` はプロジェクト内のすべてのソースの鮮度情報を計算します。これらのソースのサブセットの鮮度情報をスナップショットするには、`--select` フラグを使用します。

```bash
# Snapshot freshness for all Snowplow tables:
$ dbt source freshness --select "source:snowplow"

# Snapshot freshness for a particular source table:
$ dbt source freshness --select "source:snowplow.event"
```

### ソース鮮度出力の設定

`dbt source freshness` が完了すると、ソースの鮮度に関する情報を含む <Term id="json" /> ファイルが `target/sources.json` に保存されます。`sources.json` の例は以下のとおりです:

<File name='target/sources.json'>

```json
{
    "meta": {
        "generated_at": "2019-02-15T00:53:03.971126Z",
        "elapsed_time": 0.21452808380126953
    },
    "sources": {
        "source.project_name.source_name.table_name": {
            "max_loaded_at": "2019-02-15T00:45:13.572836+00:00Z",
            "snapshotted_at": "2019-02-15T00:53:03.880509+00:00Z",
            "max_loaded_at_time_ago_in_s": 481.307673,
            "state": "pass",
            "criteria": {
                "warn_after": {
                    "count": 12,
                    "period": "hour"
                },
                "error_after": {
                    "count": 1,
                    "period": "day"
                }
            }
        }
    }
}

```

</File>

この `sources.json` ファイルの宛先を上書きするには、`-o` (または `--output`) フラグを使用します:

```
# Output source freshness info to a different path
$ dbt source freshness --output target/source_freshness.json
```

### ソース鮮度の活用

ソース鮮度のスナップショットは、以下の点を把握するために使用できます。

1. 特定のデータソースが遅延状態にあるかどうか
2. データソース鮮度の経時的な傾向

このコマンドは手動で実行することで、いつでもソースデータの鮮度を確認できます。また、このコマンドをスケジュールに従って実行し、鮮度スナップショットの結果を定期的に保存することをお勧めします。これらの長期的なスナップショットにより、ソースデータの鮮度に関するSLA違反が発生した場合にアラートを受け取ったり、鮮度の経時的な傾向を把握したりすることが可能になります。

dbt Cloud を使用すると、スケジュールに従ってソース鮮度のスナップショットを簡単に取得できます。また、プロジェクトで定義されているすべてのソースの鮮度状態を示すダッシュボードがすぐに使用できます。dbt Cloud での鮮度のスナップショット取得の詳細については、[ドキュメント](/docs/build/sources#source-data-freshness) をご覧ください。
