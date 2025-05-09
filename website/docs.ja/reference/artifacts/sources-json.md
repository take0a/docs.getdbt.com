---
title: "Sources JSON file"
sidebar_label: "Sources"
---

**現在のスキーマ:** [`v3`](https://schemas.getdbt.com/dbt/sources/v3/index.html)

**生成元:** [`source freshness`](/reference/commands/source)

このファイルには、[フレッシュネスチェックが行われたソース](/docs/build/sources#checking-source-freshness)に関する情報が含まれています。現在、dbt Cloud はこのファイルを使用して、[ソースフレッシュネスの可視化](/docs/build/sources#source-data-freshness)を実現しています。

### 最上位キー

- [`metadata`](/reference/artifacts/dbt-artifacts#common-metadata)
- `elapsed_time`: 呼び出し時間の合計（秒）。
- `results`: フレッシュネスチェック実行の詳細の配列。

`results` 内の各エントリは、以下のキーを持つ辞書です。

- `unique_id`: 結果を [manifest](/reference/artifacts/manifest-json) 内の `sources` にマッピングする、一意のソースノード識別子。
- `max_loaded_at`: クエリ実行時のソース <Term id="table" /> 内の `loaded_at_field` タイムスタンプの最大値。
- `snapshotted_at`: クエリ実行時の現在のタイムスタンプ。
- `max_loaded_at_time_ago_in_s`: `max_loaded_at` と `snapshotted_at` の間の間隔。タイムゾーンの複雑さに対応するため、Python で計算されます。
- `criteria`: プロジェクトで定義された、このソースの鮮度しきい値。
- `status`: CLI で報告される、`max_loaded_at_time_ago_in_s` + `criteria` に基づく、このソースの鮮度ステータス。クエリが成功した場合は `pass`、`warn`、`error` のいずれか、クエリが失敗した場合は `runtime error` のいずれかになります。
- `execution_time`: このソースの鮮度チェックに費やされた合計時間
- `timing`: 実行時間をステップ (`compile` + `execute`) に分解した配列

import RowsAffected from '/snippets.ja/_run-result.md'; 

<RowsAffected/>

