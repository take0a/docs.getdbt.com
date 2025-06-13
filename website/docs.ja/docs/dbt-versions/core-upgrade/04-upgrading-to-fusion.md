---
title: "Upgrading to the dbt Fusion engine (v2.0)"
id: upgrading-to-fusion
description: New features and changes in Fusion
displayed_sidebar: "docs"
---

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';

<FusionBeta />

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />

## アップグレード前に知っておくべきこと

<Constant name="core" /> と dbt Fusion は、共通の言語仕様（プロジェクト内のコード）を共有しています。dbt Labs は、可能な限り <Constant name="core" /> と同等の機能を提供することに尽力しています。

同時に、この機会を利用して、非推奨の機能を削除し、混乱を招く動作を合理化し、誤った入力に対するより厳格な検証を提供することで、フレームワークを強化したいと考えています。そのため、既存の dbt プロジェクトを Fusion に対応させるには、ある程度の作業が必要になります。

その作業については以下に記載しています。これはシンプルでわかりやすく、多くの場合、[`dbt-autofix`](https://github.com/dbt-labs/dbt-autofix) ヘルパーで自動的に修正できるはずです。

### 白紙の状態から

dbt Labs は Fusion の開発を推し進めることに注力しており、廃止予定の機能は一切サポートしません。
- 新しいエンジンにアップグレードする前に、すべての [廃止予定の警告](https://docs.getdbt.com/reference/deprecations) を解決してください。これには、過去の廃止予定機能と [dbt Core v1.10 以降の新しい機能](https://docs.getdbt.com/docs/dbt-versions/core-upgrade/upgrading-to-v1.10#deprecation-warnings) が含まれます。_Fusion がベータ版の間は検証警告が表示されますが、Fusion がプレビュー版に移行した時点でこれらの警告はエラーになります。_
- すべての [動作変更フラグ](https://docs.getdbt.com/reference/global-configs/behavior-changes#behaviors) は削除されます（通常は有効になります）。 `dbt_project.yml` で `flags:` を使用してこれらをオプトアウトすることはできなくなりました。

### エコシステムパッケージ

最も人気のある `dbt-labs` パッケージ (`dbt_utils`、`audit_helper`、`dbt_external_tables`、`dbt_project_evaluator`) は既に Fusion と互換性があります。dbt 以外の組織によって公開されている外部パッケージは、古いコードや互換性のない機能を使用している可能性があり、新しい Fusion エンジンで解析できない可能性があります。Fusion のベータ版を発表したので、他のパッケージメンテナーと協力して、Fusion の準備と作業を進めていきます。人気のあるパッケージで Fusion との互換性を保つために新しいリリースへのアップグレードが必要であることが判明した場合は、ここでその旨を文書化します。

### 機能変更

Fusionエンジンの開発中、dbtフレームワークを改善する機会がありました。具体的には、可能な場合は早期にエラーを検知する、バグを修正する、実行順序を最適化する、不要になったフラグを廃止するといった改善が行われました。その結果、既存の動作にいくつかの具体的かつ微妙な変更が加えられました。

Fusionにアップグレードすると、以下の機能変更が予想されます。

#### リレーションの解析時出力では、空文字列ではなく完全修飾名が出力されます。

dbt Core v1 では、`get_relation()` の結果を出力する際、その Jinja の解析時出力には `None` が出力されていました（未定義オブジェクトは文字列「None」に強制変換されます）。

Fusion では、`get_relation()` 呼び出しのインテリジェントなバッチ処理（および `dbt compile` の大幅な高速化）を支援するために、dbt は `get_relation()` アダプタ呼び出し用に、解析時に解決された完全修飾名を持つリレーションオブジェクトを構築する必要があります。

Fusion で完全修飾名を持つリレーションオブジェクトを構築すると、`p​​rint()`、`log()`、または解析時に `stdout` または `stderr` に出力するすべての Jinja マクロにおいて、dbt Core v1 とは異なる動作が発生します。

Example:

```jinja
{% set relation = adapter.get_relation(
database=db_name,
schema=db_schema,
identifier='a')
%}
{{ print('relation: ' ~ relation) }}

{% set relation_via_api = api.Relation.create(
database=db_name,
schema=db_schema,
identifier='a'
) %}
{{ print('relation_via_api: ' ~ relation_via_api) }}
```

dbt Core v1 での `dbt parse` 後の出力:

```
relation: None
relation_via_api: my_db.my_schema.my_table
```

Fusion での `dbt parse` 後の出力:

```
relation: my_db.my_schema.my_table
relation_via_api: my_db.my_schema.my_table
```

#### 非推奨のフラグ

dbt Core v1 で使用されていた一部のフラグは、Fusion では機能しなくなります。これらのフラグを Fusion で使用して dbt コマンドに渡した場合、コマンドはエラーにはなりませんが、フラグ自体は何も実行されません（それに応じて警告が表示されます）。

このルールには例外が 1 つあります。`--models` / `--model` / `-m` フラグは、dbt Core v0.21 (2021 年 10 月) で `--select` / `--s` に名前が変更されました。このフラグを暗黙的に指定しないと、コマンドの選択基準が無視され、小さなサブセットのみを選択するつもりが DAG 全体を構築してしまう可能性があります。そのため、`--models` / `--model` / `-m` フラグは Fusion で**エラー** を発生させます。ジョブ定義を適宜更新してください。

| flag name | remediation |
| ----------| ----------- |
| `dbt seed` [`--show`](https://docs.getdbt.com/reference/commands/seed) | N/A |
| [`--print` / `--no-print`](https://docs.getdbt.com/reference/global-configs/print-output) | No action required |
| [`--printer-width`](https://docs.getdbt.com/reference/global-configs/print-output#printer-width) | No action required |
| [`--source`](https://docs.getdbt.com/reference/commands/deps#non-hub-packages) | No action required |
| [`--record-timing-info` / `-r`](https://docs.getdbt.com/reference/global-configs/record-timing-info) | No action required |
| [`--cache-selected-only` / `--no-cache-selected-only`](https://docs.getdbt.com/reference/global-configs/cache) | No action required |
| [`--clean-project-files-only` / `--no-clean-project-files-only`](https://docs.getdbt.com/reference/commands/clean#--clean-project-files-only) | No action required |
| `--single-threaded` / `--no-single-threaded` | No action required |
| `dbt source freshness` [`--output` / `-o`](https://docs.getdbt.com/docs/deploy/source-freshness)  | |
| [`--config-dir`](https://docs.getdbt.com/reference/commands/debug)  | No action required | 
| [`--add-package`](https://docs.getdbt.com/reference/commands/deps) | No action required |
| [`--resource-type` / `--exclude-resource-type`](https://docs.getdbt.com/reference/global-configs/resource-type) | change to `--resource-types` / `--exclude-resource-types` |
| `--show-resource-report` / `--no-show-resource-report` | No action required |
| [`--log-cache-events` / `--no-log-cache-events`](https://docs.getdbt.com/reference/global-configs/logs#logging-relational-cache-events) | No action required | 
| `--use-experimental-parser` / `--no-use-experimental-parser` | No action required |
| [`--empty-catalog`](https://docs.getdbt.com/reference/commands/cmd-docs#dbt-docs-generate ) | |
| [`--compile` / `--no-compile`](https://docs.getdbt.com/reference/commands/cmd-docs#dbt-docs-generate) | |
| `--inline-direct` |  No action required |
| `--partial-parse-file-diff` / `--no-partial-parse-file-diff` | No action required |
| `--partial-parse-file-path` | No action required |
| `--populate-cache` / `--no-populate-cache` | No action required |
| `--static-parser` / `--no-static-parser` | No action required |
| `--use-fast-test-edges` / `--no-use-fast-test-edges` | No action required |
| [`--introspect` / `--no-introspect`](https://docs.getdbt.com/reference/commands/compile#introspective-queries) | No action required |
| `--inject-ephemeral-ctes` / `--no-inject-ephemeral-ctes` | | 
| [`--partial-parse` / `--no-partial-parse`](https://docs.getdbt.com/reference/parsing#partial-parsing)  | No action required |

#### ローカルパッケージがハブパッケージに依存し、そのハブパッケージをルートパッケージも必要としている場合、パッケージバージョンの競合によりエラーが発生します。

ローカルパッケージがハブパッケージに依存し、そのハブパッケージをルートパッケージも必要としている場合、dbt Core v1 では `dbt deps` は競合するバージョンを解決せず、ルートプロジェクトが要求したものをすべてインストールします。

Fusion で次のエラーが表示されます。

```bash
error: dbt8999: Cannot combine non-exact versions: =0.8.3 and =1.1.1
```


#### 存在しないマクロ呼び出しとアダプタメソッドでは解析が失敗します。

dbt で存在しないマクロを呼び出す場合:

```sql
select
  id as payment_id,
  # my_nonexistent_macro is a macro that DOES NOT EXIST
  {{ my_nonexistent_macro('amount') }} as amount_usd,
from app_data.payments
```

または存在しないアダプターメソッド:

```sql
{{ adapter.does_not_exist() }}
```

dbt Core v1 では、`dbt parse` は成功しますが、`dbt compile` は失敗します。

Fusion は `parse` 中にエラーを出力します。

#### ジェネリックテストがない場合、解析は失敗します。

プロジェクトに未定義のジェネリックテストがある場合:

```yaml

models:
  - name: dim_wizards
    tests:
      - does_not_exist

```

dbt Core v1 では、`dbt parse` は成功しますが、`dbt compile` は失敗します。

Fusion は `parse` 中にエラーを出力します。

#### 変数が見つからない場合、解析は失敗します。

プロジェクト内に未定義の変数がある場合:

```sql

select {{ var('does_not_exist') }} as my_column

```

dbt Core v1 では、`dbt parse` は成功しますが、`dbt compile` は失敗します。

Fusion は `parse` 中にエラーを出力します。

#### 旧バージョンのマニフェストのサポート終了

以下の場合、dbt-core 1.8 より前のバージョンとの相互運用はできなくなります。
- Fusion と旧バージョン（v1.8 より前）の dbt Core をハイブリッドでご利用のお客様
- 旧バージョン（v1.8 より前）の dbt Core から Fusion にアップグレードするお客様

Fusion は、`state:modified` 比較の遅延などの機能を実現する旧マニフェストとは相互運用できません。

#### `dbt clean` は、設定されたリソースパス内のファイルやプロジェクトディレクトリ外のファイルは削除しません。

dbt Core v1 では、`dbt clean` は次のものを削除します。
- `clean-targets` が絶対パスまたは `../` を含む相対パスで設定されている場合、プロジェクトディレクトリ外のすべてのファイル。ただし、これを無効にするオプトイン構成（`--clean-project-files-only` / `--no-clean-project-files-only`）があります。
- `asset-paths` または `doc-paths` 内のすべてのファイル（`model-paths` や `seed-paths` などの他のリソースパスは制限されていますが）。

Fusion では、`dbt clean` は、設定されたリソースパス内のファイルやプロジェクトディレクトリ外のファイルを削除しません。

#### すべてのユニットテストは最初に `dbt build` で実行されます。

dbt Core v1 では、必要な列名と型情報を取得するために、ユニットテスト対象のモデルの直接の親がウェアハウス内に存在している必要がありました。`dbt build` は、ユニットテスト（および依存モデル）を_系統順_で実行します。

Fusion では、`dbt build` は組み込みの列名と型認識機能により、最初にすべてのユニットテストを実行し、その後 DAG の残りの部分をビルドします。

#### `--threads` の設定

dbt Core はデフォルトで `--threads 1` で実行されます。この数値を増やすことで、リモートデータプラットフォーム上でより多くのノードを並列実行できます。最大並列処理数は、DAG で有効になっている最大並列処理数までです。

Fusion では、`--threads` が設定されていない場合、または `--threads 0` に設定されている場合、dbt は最大スレッド数としてアダプタごとのデフォルト値を使用します。データプラットフォームによっては、他のプラットフォームよりも多くの同時接続を処理できる場合があります。`--threads` にユーザーが設定した値（CLI フラグまたは `profiles.yml` 経由）がある場合、Fusion はその値を使用します。

#### コンパイルエラー発生後も無関係なノードのコンパイルを継続

dbt Core の `compile` がモデルの 1 つをコンパイル中にエラーを検出すると、dbt は直ちに停止し、他のノードのコンパイルは行いません。

Fusion の `compile` がエラーを検出すると、コンパイルに失敗したノードの下流ノードはスキップされますが、DAG の残りのノードは（設定されたスレッド数または最適なスレッド数まで並列に）コンパイルを継続します。

#### シードに余分なカンマがあっても、列は追加されません。

dbt Core v1 では、シードに余分なカンマがある場合、dbt は空の列を追加したシードを作成します。

例えば、次のシードファイル（余分なカンマあり）は、

```
animal,  
dog,  
cat,  
bear,  

```

`dbt seed` を実行すると、次のテーブルが生成されます:

| animal | b |  
| ------ | - |  
| dog    |   |  
| cat    |   |  
| bear   |   |  

Fusion は、`dbt seed` の結果のテーブルにこの追加の列を生成しません:

| animal |  
| ------ |  
| dog    |  
| cat    |  
| bear   |  
