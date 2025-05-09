---
title: "Manifest JSON file"
sidebar_label: "Manifest"
---

import ManifestVersions from '/snippets.ja/_manifest-versions.md';

<ManifestVersions />

**生成元:** プロジェクトを解析する任意のコマンド。[`deps`](/reference/commands/deps)、[`clean`](/reference/commands/clean)、[`debug`](/reference/commands/debug)、[`init`](/reference/commands/init)を **除く** すべてのコマンドが含まれます。

このファイルには、dbtプロジェクトのリソース（モデル、テスト、マクロなど）の完全な表現が含まれており、すべてのノード構成とリソースプロパティが含まれます。一部のモデルやテストのみを実行している場合でも、すべてのリソースが（無効化されていない限り）ほとんどのプロパティと共にマニフェストに表示されます。 （`compiled_sql` など、一部のノードプロパティは実行されたノードにのみ表示されます。）

現在、dbt はこのファイルを使用して [ドキュメントサイト](/docs/collaborate/build-and-view-your-docs) に情報を入力し、[状態の比較](/reference/node-selection/syntax#about-node-selection) を実行します。コミュニティのメンバーは、このファイルを使用して、説明とテストを持つモデルの数を確認しています。

### 最上位キー

- [`metadata`](/reference/artifacts/dbt-artifacts#common-metadata)
- `nodes`: すべての分析、モデル、シード、スナップショット、テストの辞書。
- `sources`: ソースの辞書。
- `metrics`: メトリクスの辞書。
- `exposures`: エクスポージャーの辞書。
- `groups`: グループの辞書。(**注:** v1.5 で追加)
- `macros`: マクロの辞書。
- `docs`: `docs` ブロックの辞書。
- `parent_map`: 各リソースの第一階層の親を含む辞書。
- `child_map`: 各リソースの第一階層の子を含む辞書。
- `group_map`: グループ名とリソースノードをマッピングする辞書。
- `selectors`: [YAML `selectors`](/reference/node-selection/yaml-selectors) の拡張辞書表現。
- `disabled`: `enabled: false` が指定されたリソースの配列。

### リソースの詳細

`nodes`、`sources`、`metrics`、`exposures`、`macros`、`docs` 内にネストされたすべてのリソースは、以下の基本プロパティを持ちます。

- `name`: リソース名。
- `unique_id`: `<resource_type>.<package>.<resource_name>`。辞書キーと同じ。
- `package_name`: このリソースを定義するパッケージ名。
- `root_path`: このリソースのパッケージの絶対ファイルパス。(**注:** ノード間の重複情報を削減するため、dbt Core v1.4 / manifest v8 ではほとんどのノードタイプで削除されましたが、シードでは引き続き存在します。)
- `path`: このリソースの定義の「リソースパス」(`model-paths`、`seed-paths` など) 内での相対ファイルパス。
- `original_file_path`: このリソース定義の相対ファイルパス（リソースパスを含む）。

各リソースには、リソースタイプに関連するいくつかの追加プロパティがあります。

### dbt JSON スキーマ

dbt で生成されたアーティファクトの記述と使用方法については、[dbt JSON スキーマ](https://schemas.getdbt.com/) を参照してください。

**注**: `manifest.json` のバージョン番号は dbt のバージョンと関連していますが、必ずしも同じではありません。そのため、dbt のバージョンに合った正しい `manifest.json` のバージョンを使用する必要があります。正しい `manifest.json` のバージョンを確認するには、上部のナビゲーションで dbt のバージョン (`v1.5` など) を選択します。

[このページ](/reference/artifacts/manifest-json) の冒頭にある表を参照して、マニフェストのバージョンと dbt のバージョンがどのように一致するかを理解してください。
