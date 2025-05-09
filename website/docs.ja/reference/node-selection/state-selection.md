---
title: "dbt の state について"
description: "dbt 操作はステートレスかつべき等ですが、アーティファクトにより、スリム CI や延期などの状態ベースの機能が有効になります。"
pagination_next: "reference/node-selection/configure-state"
---

dbt に関する最大の前提の一つは、その操作が **ステートレス** かつ **<Term id="idempotent" />** であるべきであるということです。つまり、モデルが過去に何回実行されたか、あるいはそもそも実行されたことがあるかどうかは関係ありません。1 回実行しても 1000 回実行しても問題ありません。同じ生データであれば、同じ変換結果が期待できます。dbt の実行は、他の実行について「知る」必要はありません。必要なのは、プロジェクト内のコードと、データベース内のオブジェクトが _今_ どのような状態であるかだけです。

ただし、dbt は「状態」、つまりプロジェクト リソース（ノードとも呼ばれます）、データベース オブジェクト、および呼び出し結果の詳細な特定時点のビューを [アーティファクト](/docs/deploy/artifacts) の形式で保存します。必要に応じて、dbt はこれらのアーティファクトを使用して特定の操作を通知できます。重要なのは、操作自体は依然としてステートレスであり、<Term id="idempotent" /> であるということです。つまり、同じマニフェストと同じ生データがあれば、dbt は同じ変換結果を生成します。

dbt は、`--state` フラグにファイルパスが渡されている限り、以前の呼び出しの成果物を利用できます。これは以下の前提条件となります。
- [`state` セレクター](/reference/node-selection/methods#state)。これにより、dbt は現在のプロジェクトのコードと状態マニフェストを比較することで、新規または変更されたリソースを識別できます。
- [別の環境への延期](/reference/node-selection/defer)。これにより、dbt は現在の環境に存在しない上流の選択されていないリソースを識別し、それらの参照を状態マニフェストによって提供される環境に「延期」できます。
- [`dbt clone` コマンド](/reference/commands/clone)。これにより、dbt は `--state` フラグに指定されたマニフェスト内の位置に基づいてノードを複製できます。

[`state`](/reference/node-selection/methods#state) セレクターと deferral を組み合わせることで、["slim CI"](/best-practices/best-practice-workflows#run-only-modified-models-to-test-changes-slim-ci) が実現します。今後のリリースでは、`--state` フラグに渡されるアーティファクトを活用できる機能をさらに追加する予定です。

## 関連ドキュメント
- [状態選択の設定](/reference/node-selection/configure-state)
- [状態比較に関する注意事項](/reference/node-selection/state-comparison-caveats)
