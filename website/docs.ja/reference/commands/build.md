---
title: "dbt build コマンドについて"
sidebar_label: "build"
id: "build"
---

`dbt build` コマンドは、以下の処理を実行します。
- run models
- test tests
- snapshot snapshots
- seed seeds

選択したリソースまたはプロジェクト全体に対して、DAG 順に実行します。

## 詳細

**アーティファクト:** `build` タスクは、1 つの [マニフェスト](/reference/artifacts/manifest-json) と 1 つの [実行結果アーティファクト](/reference/artifacts/run-results-json) を出力します。実行結果には、ビルド対象として選択されたすべてのモデル、テスト、シード、スナップショットに関する情報が 1 つのファイルにまとめられます。

**失敗時のスキップ:** 上流リソースのテストは下流リソースの実行をブロックし、テストが失敗すると、それらの下流リソースは完全にスキップされます。例: `model_b` が `model_a` に依存しており、`model_a` の `unique` テストが失敗した場合、`model_b` は `SKIP` されます。
- テストによってスキップが発生しないようにしたいですか？ [重大度またはしきい値](/reference/resource-configs/severity) を `error` ではなく `warn` に調整します。
- 複数の親を持つテストで、一方の親がもう一方の親に依存している場合（例: `model_a` と `model_b` 間の `relationships` テスト）、そのテストは最下流の親（`model_b`）の子のみをブロックしてスキップします。

**リソースの選択:** `build` タスクは、標準的な選択構文（`--select`、`--exclude`、`--selector`）と、最終フィルターを提供する `--resource-type` フラグ（`list` と同様に）をサポートしています。どのリソースが選択されても、`build` はそれらのリソースに対して実行/テスト/スナップショット/シードを行います。
- テストは間接選択をサポートしているため、`dbt build -s model_a` は `model_a` の実行とテストの両方を実行します。これはどういう意味でしょうか？`model_a` に直接依存するテストはすべて含まれますが、それらのテストが他の選択されていない親にも依存していない限りです。詳細と例については、[テストの選択](/reference/node-selection/test-selection-examples) を参照してください。

**フラグ:** `build` タスクは、`run`、`test`、`snapshot`、`seed` と同じフラグをすべてサポートしています。複数のタスク間で共有されるフラグ（例: `--full-refresh`）の場合、`build` は選択されたすべてのリソースタイプに対して同じ値を使用します（例: モデルとシードの両方がフルリフレッシュされます）。

<VersionBlock firstVersion="1.8">

### `--empty` フラグ

`build` コマンドは、スキーマのみのドライランを構築するための `--empty` フラグをサポートしています。`--empty` フラグは、参照とソースを 0 行に制限します。dbt はターゲットデータウェアハウスに対してモデル SQL を実行しますが、入力データの高コストな読み取りを回避します。これにより依存関係が検証され、モデルが適切に構築されることが保証されます。

import SQLCompilationError from '/snippets.ja/_render-method.md';

<SQLCompilationError />

## テスト

ユニットテストを適用して `dbt build` を実行すると、モデルは系統と依存関係に基づいて処理されます。テストは次のように実行されます。

- [ユニットテスト](/docs/build/unit-tests) は SQL モデルに対して実行されます。
- モデルがマテリアライズされます。
- [データテスト](/docs/build/data-tests) はモデルに対して実行されます。

ユニットテストが正常に完了した場合にのみモデルがマテリアライズされるため、ウェアハウスのコストを削減できます。

ユニットテストとデータテストは、`dbt build` で `--select test_type:unit` または `--select test_type:data` を使用して選択できます（`--exclude` フラグも同様です）。

</VersionBlock>

### 例


```
$ dbt build
Running with dbt=1.9.0-b2
Found 1 model, 4 tests, 1 snapshot, 1 analysis, 341 macros, 0 operations, 1 seed file, 2 sources, 2 exposures

18:49:43 | Concurrency: 1 threads (target='dev')
18:49:43 |
18:49:43 | 1 of 7 START seed file dbt_jcohen.my_seed............................ [RUN]
18:49:43 | 1 of 7 OK loaded seed file dbt_jcohen.my_seed........................ [INSERT 2 in 0.09s]
18:49:43 | 2 of 7 START view model dbt_jcohen.my_model.......................... [RUN]
18:49:43 | 2 of 7 OK created view model dbt_jcohen.my_model..................... [CREATE VIEW in 0.12s]
18:49:43 | 3 of 7 START test not_null_my_seed_id................................ [RUN]
18:49:43 | 3 of 7 PASS not_null_my_seed_id...................................... [PASS in 0.05s]
18:49:43 | 4 of 7 START test unique_my_seed_id.................................. [RUN]
18:49:43 | 4 of 7 PASS unique_my_seed_id........................................ [PASS in 0.03s]
18:49:43 | 5 of 7 START snapshot snapshots.my_snapshot.......................... [RUN]
18:49:43 | 5 of 7 OK snapshotted snapshots.my_snapshot.......................... [INSERT 0 5 in 0.27s]
18:49:43 | 6 of 7 START test not_null_my_model_id............................... [RUN]
18:49:43 | 6 of 7 PASS not_null_my_model_id..................................... [PASS in 0.03s]
18:49:43 | 7 of 7 START test unique_my_model_id................................. [RUN]
18:49:43 | 7 of 7 PASS unique_my_model_id....................................... [PASS in 0.02s]
18:49:43 |
18:49:43 | Finished running 1 seed, 1 view model, 4 tests, 1 snapshot in 1.01s.

Completed successfully

Done. PASS=7 WARN=0 ERROR=0 SKIP=0 TOTAL=7
```
