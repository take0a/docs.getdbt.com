---
title: "Syntax 概要"
description: "ノード選択構文を使用すると、特定のモデルおよびリソースに対して dbt コマンドを実行できます。"
---

dbt のノード選択構文により、特定のリソースのみを dbt の呼び出し時に実行することが可能になります。この選択構文は、以下のサブコマンドで使用されます:

| command                         | argument(s)                                                          |
| :------------------------------ | -------------------------------------------------------------------- |
| [run](/reference/commands/run)             | `--select`, `--exclude`, `--selector`, `--defer`                     |
| [test](/reference/commands/test)           | `--select`, `--exclude`, `--selector`, `--defer`                     |
| [seed](/reference/commands/seed)           | `--select`, `--exclude`, `--selector`                                |
| [snapshot](/reference/commands/snapshot)   | `--select`, `--exclude`  `--selector`                                |
| [ls (list)](/reference/commands/list)      | `--select`, `--exclude`, `--selector`, `--resource-type`             |
| [compile](/reference/commands/compile)     | `--select`, `--exclude`, `--selector`, `--inline`                    |
| [freshness](/reference/commands/source)    | `--select`, `--exclude`, `--selector`                                |
| [build](/reference/commands/build)         | `--select`, `--exclude`, `--selector`, `--resource-type`, `--defer`  |
| [docs generate](/reference/commands/cmd-docs) | `--select`, `--exclude`, `--selector`                  |

:::info ノードとリソース

<a href="https://en.wikipedia.org/wiki/Vertex_(graph_theory)">「ノード」</a>と「リソース」という用語は同じ意味で使用します。これらは、プロジェクト内のすべてのモデル、テスト、ソース、シード、スナップショット、エクスポージャー、分析を包含します。これらは、dbt の DAG (有向非巡回グラフ) を構成するオブジェクトです。
:::

`--select` 引数と `--selector` 引数は、どちらもリソースを選択できるという点で似ています。違いを理解するには、[`--select` と `--selector` の違い](/reference/node-selection/yaml-selectors#difference-between---select-and---selector) を参照してください。

## リソースの指定

デフォルトでは、`dbt run` は依存関係グラフ内のすべてのモデルを実行します。`dbt seed` はすべてのシードを作成し、`dbt snapshot` はすべてのスナップショットを実行します。`--select` フラグは、実行するノードのサブセットを指定するために使用されます。

[POSIX 標準](https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/V1_chap12.html) に準拠し、理解を容易にするために、CLI ユーザーは `--select` または `--exclude` オプションに引数を渡す際に引用符を使用することをお勧めします（単一または複数のスペース区切り、またはカンマ区切りの引数を含む）。引用符を使用しない場合、すべてのオペレーティングシステム、端末、およびユーザーインターフェースで確実に動作しない可能性があります。たとえば、`dbt run --select "my_dbt_project_name"` は、プロジェクト内のすべてのモデルを実行します。

### 選択はどのように機能しますか？

1. dbt は、`--select` 条件の 1 つ以上に一致するすべてのリソースを、まず [選択方法](/reference/node-selection/methods) (例: `tag:`)、次に [グラフ演算子](/reference/node-selection/graph-operators) (例: `+`)、最後に集合演算子 ([結合](/reference/node-selection/set-operators#unions)、[交差](/reference/node-selection/set-operators#intersections)、[除外](/reference/node-selection/exclude)) の順に収集します。

2. 選択されるリソースは、モデル、ソース、シード、スナップショット、テストです。 （テストは親を介して「間接的に」選択することもできます。詳細については、[テスト選択の例](/reference/node-selection/test-selection-examples)を参照してください。）

3. dbt は、選択中の様々なタイプのリソースのリストを保持します。最終ステップとして、現在のタスクのリソースタイプと一致しないリソースをすべて破棄します。（`dbt seed` の場合はシードのみ、`dbt run` の場合はモデルのみ、`dbt test` の場合はテストのみ、といった具合です。）

## ショートカット

ビルド（実行、テスト、シード、スナップショット）するリソースを選択、または最新かどうかをチェック: `--select`, `-s`

### 例

デフォルトでは、`dbt run` は依存関係グラフ内のすべてのモデルを実行します。開発（およびデプロイ）時には、実行するモデルのサブセットのみを指定すると便利です。`dbt run` で `--select` フラグを使用すると、実行するモデルのサブセットを選択できます。以下の引数（`--select`、`--exclude`、`--selector`）は、`test` や `build` などの他の dbt タスクにも適用されることに注意してください。

<Tabs>
<TabItem value="select" label="Examples of select flag">

`--select` フラグは1つ以上の引数を受け入れます。各引数は以下のいずれかになります。

1. パッケージ名
2. モデル名
3. モデルディレクトリへの完全修飾パス
4. 選択方法 (`path:`, `tag:`, `config:`, `test_type:`, `test_name:`)

例:

```bash
dbt run --select "my_dbt_project_name"   # runs all models in your project
dbt run --select "my_dbt_model"          # runs a specific model
dbt run --select "path/to/my/models"     # runs all models in a specific directory
dbt run --select "my_package.some_model" # run a specific model in a specific package
dbt run --select "tag:nightly"           # run models with the "nightly" tag
dbt run --select "path/to/models"        # run models contained in path/to/models
dbt run --select "path/to/my_model.sql"  # run a specific model by its path
```

</TabItem>

<TabItem value="subset" label="Examples of subsets of nodes">

dbt は、ノードのサブセットを定義するための短縮形言語をサポートしています。この言語では、以下の文字を使用します。

- プラス演算子 [(`+`)](/reference/node-selection/graph-operators#the-plus-operator)
- アット演算子 [(`@`)](/reference/node-selection/graph-operators#the-at-operator)
- アスタリスク演算子 (`*`)
- カンマ演算子 (`,`)

例:

```bash
# multiple arguments can be provided to --select
dbt run --select "my_first_model my_second_model"

# select my_model and all of its children
dbt run --select "my_model+"     

# select my_model, its children, and the parents of its children
dbt run --models @my_model          

# these arguments can be projects, models, directory paths, tags, or sources
dbt run --select "tag:nightly my_model finance.base.*"

# use methods and intersections for more complex selectors
dbt run --select "path:marts/finance,tag:nightly,config.materialized:table"
```

</TabItem>

</Tabs>

選択ロジックが複雑になり、コマンドライン引数として入力するのが難しくなってきたら、
[yaml セレクター](/reference/node-selection/yaml-selectors) の使用を検討してください。`--selector` フラグを使用すると、定義済みのセレクターを使用できます。
`--selector` を使用する場合、他のほとんどのフラグ（具体的には `--select` と `--exclude`）は無視されることに注意してください。

`--select` 引数と `--selector` 引数は、どちらもリソースを選択できるという点で似ています。`--select` 引数と `--selector` 引数の違いについては、[このセクション](/reference/node-selection/yaml-selectors#difference-between---select-and---selector) で詳細をご覧ください。

### `ls` コマンドによるトラブルシューティング

選択構文の構築とデバッグは難しい場合があります。選択されるノードの「プレビュー」を取得するには、[`list` コマンド](/reference/commands/list) の使用をお勧めします。このコマンドを選択構文と組み合わせると、選択条件を満たすノードのリストが出力されます。`dbt ls` コマンドは、あらゆる種類の選択構文引数をサポートしています。例:

```bash
dbt ls --select "path/to/my/models" # Lists all models in a specific directory.
dbt ls --select "source_status:fresher+" # Shows sources updated since the last dbt source freshness run.
dbt ls --select state:modified+ # Displays nodes modified in comparison to a previous state.
dbt ls --select "result:<status>+" state:modified+ --state ./<dbt-artifact-path> # Lists nodes that match certain [result statuses](/reference/node-selection/syntax#the-result-status) and are modified.
```

<Snippet path="discourse-help-feed-header" />
<DiscourseHelpFeed tags="node-selection"/>
