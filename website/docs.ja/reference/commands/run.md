---
title: "dbt run コマンドについて"
sidebar_label: "run"
description: "dbt run コマンドは、コンパイルされた SQL モデルをターゲット データベースに対して実行します。"
id: "run"
---

## 概要

`dbt run` は、コンパイル済みの SQL モデルファイルを現在の `target` データベースに対して実行します。dbt はターゲットデータベースに接続し、指定された <Term id="materialization" /> 戦略を使用して、すべてのデータモデルをマテリアライズするために必要な SQL を実行します。
モデルは、コンパイル時に生成された依存関係グラフで定義された順序で実行されます。インテリジェントなマルチスレッド処理により、依存関係に違反することなく実行時間を最小限に抑えることができます。

新しいモデルをデプロイする際には、多くの場合、以前のバージョンのモデルを破棄する必要があります。
このような場合、`dbt run` は、まず各モデルを一時的な名前で構築し、次に既存のモデルを削除して正しい名前に変更することで、モデルが使用できない時間を最小限に抑えます。
トランザクションをサポートするデータベースアダプタの場合、削除と名前の変更は単一のデータベーストランザクション内で実行されます。

## 増分モデルの更新

`dbt run` に `--full-refresh` フラグを指定すると、dbt は増分モデルを <Term id="table" /> モデルとして扱います。これは、次の場合に役立ちます。

1. 増分モデルのスキーマが変更され、再作成する必要がある場合。
2. モデルコードに新しいロジックが追加されたため、増分モデル全体を再処理する必要がある場合。

<File name='bash'>

```shell
dbt run --full-refresh
```

</File>

このフラグは、短縮名「dbt run -f」でも指定できます。

dbt コンパイルコンテキストでは、このフラグは [flags.FULL_REFRESH](/reference/dbt-jinja-functions/flags) として使用できます。さらに、`--full-refresh` フラグが指定されている場合、`is_incremental()` マクロは、*すべての*モデルに対して `false` を返します。

<File name='models/example.sql'>

```sql
select * from all_events

-- if the table already exists and `--full-refresh` is
-- not set, then only add new records. otherwise, select
-- all records.
{% if is_incremental() %}
   where collector_tstamp > (
     select coalesce(max(max_tstamp), '0001-01-01') from {{ this }}
   )
{% endif %}
```

</File>

## 特定のモデルの実行

dbt では、マテリアライズする特定のモデルを選択することもできます。これは、異なる間隔で異なるモデルセットを実行したい特別なシナリオで役立ちます。また、新しいモデルの開発とテスト中にマテリアライズするテーブルを制限したい場合にも役立ちます。

詳細については、[モデル選択構文ドキュメント](/reference/node-selection/syntax) を参照してください。

特定のモデルの親または子の実行の詳細については、[グラフ演算子ドキュメント](/reference/node-selection/graph-operators) を参照してください。

## 警告をエラーとして扱う

[グローバル設定](/reference/global-configs/warnings) を参照してください

## 素早く失敗すること

[グローバル設定](/reference/global-configs/failing-fast) を参照してください

## ログの色分けを有効または無効にする

[グローバル設定](/reference/global-configs/print-output#print-color) を参照してください

<VersionBlock firstVersion="1.8">

## `--empty` フラグ

`run` コマンドは、スキーマのみのドライランを作成するための `--empty` フラグをサポートしています。`--empty` フラグは、参照とソースを 0 行に制限します。dbt はターゲットデータウェアハウスに対してモデル SQL を実行しますが、入力データの高負荷な読み取りを回避します。これにより依存関係が検証され、モデルが適切に構築されることが保証されます。

</VersionBlock>

## ステータスコード

[list_runs API](/dbt-cloud/api-v2#/operations/List%20Runs) を呼び出すと、返される実行ごとにステータスコードが返されます。使用可能な実行ステータスコードは次のとおりです。

- Starting = 1
- Running = 3
- Success = 10
- Error = 20
- Canceled = 30
- Skipped = 40
