---
title: "プロジェクト解析"
description: "dbt でのプロジェクト解析構成を理解するには、このガイドをお読みください。"
---

## 関連ドキュメント
- `dbt parse` [コマンド](/reference/commands/parse)
- 部分解析 [プロファイル設定](/docs/core/connect-data-platform/profiles.yml#partial_parse) と [CLI フラグ](/reference/global-configs/parsing)
- 解析 [CLI フラグ](/reference/global-configs/parsing)

## 解析とは？

dbt は毎回の呼び出し開始時に、プロジェクト内のすべてのファイルを読み取り、情報を抽出し、すべてのオブジェクト（モデル、ソース、マクロなど）を含むマニフェストを作成します。dbt はモデル内の `ref()`、`source()`、`config()` マクロ呼び出しを使用して、プロパティの設定、依存関係の推測、プロジェクトの DAG の構築などを行います。

プロジェクトの解析は遅くなる可能性があり、特にプロジェクトが大きくなると（モデルが数百、ファイルが数千に及ぶなど）、開発において大きな負担となります。現在、dbt のパフォーマンスを最適化する方法はいくつかあります。
- PyYAML 用の LibYAML バインディング
- 部分解析：呼び出し間で変更されていないファイルの再解析を回避
- 静的パーサー：シンプルなモデルからより高速に情報を抽出
- [RPC サーバー](/reference/commands/rpc)：マニフェストをメモリに保持し、サーバーの起動時/切断時にプロジェクトを再解析

これらの最適化を組み合わせて使用​​することで、解析時間を数分から数秒に短縮できます。ただし、それぞれに既知の制限があるため、デフォルトでは無効になっています。

## PyYAML + LibYAML

dbt は [PyYAML](https://pyyaml.org/wiki/PyYAML) を使用して、プロジェクト内の YAML ファイルを読み取り、検証します。PyYAML は純粋な Python で記述されていますが、システムで利用可能な場合は [LibYAML](https://pyyaml.org/wiki/LibYAML) (C で記述されており、はるかに高速) を活用できます。dbt はプロジェクトを解析する際に、必ず最初に LibYAML が利用可能かどうかを確認します。

dbt をインストールした環境で次のコマンドを実行すると、LibYAML がインストールされているかどうかをテストできます。

```
python -c "from yaml import CLoader"
```

## 部分解析

dbt はプロジェクトを解析した後、内部プロジェクトマニフェストを `partial_parse.msgpack` というファイルに保存します。部分解析が有効になっている場合、dbt はこの内部マニフェストを使用して、プロジェクトの最終解析以降に変更されたファイル（ある場合）を特定します。その後、変更されたファイル、またはそれらの変更に関連するファイルのみを解析します。

v1.0 以降、部分解析はデフォルトで **オン** になっています。開発環境において、部分解析を行うことで実行開始時の待機時間を大幅に短縮できるため、開発サイクルとイテレーションの高速化につながります。

[`PARTIAL_PARSE` グローバル設定](/reference/global-configs/parsing) は、`profiles.yml`、環境変数、または CLI フラグによって有効化または無効化できます。

### 既知の制限事項

解析時属性（依存関係、構成、リソースプロパティ）は、解析時コンテキストを使用して解決されます。部分解析が有効になっている場合、特定のコンテキスト変数が変更されると、それらの属性は再解決されず、古くなる可能性があります。

特に、これらの属性が [`run_started_at`](/reference/dbt-jinja-functions/run_started_at)、[`invocation_id`](/reference/dbt-jinja-functions/invocation_id)、[flags](/reference/dbt-jinja-functions/flags) などの「揮発性」コンテキスト変数に依存している場合、誤った結果が表示される可能性があります。これらの変数は、呼び出しごとに変更される可能性があります（変更されることが保証されています）。dbt Labs は、これらの変数を使用して解析時属性（依存関係、構成、リソースプロパティ）を設定することを強く推奨しません。

v1.0 以降、dbt は環境変数の変更を検出します。[`env_var`](/reference/dbt-jinja-functions/env_var) 値に依存するファイルのみを選択的に再解析します。(環境変数が `profiles.yml` または `dbt_project.yml` で使用されている場合は、完全な再解析が必要です。) ただし、dbt は環境変数を含む **説明** を再レンダリングしません。説明に頻繁に変更される環境変数が含まれている場合 (これは非常にまれですが)、ドキュメント生成時に完全な再解析を実行することをお勧めします: `dbt --no-partial-parse docs generate`。

実行間で特定の入力が変更された場合、dbt は完全な再解析をトリガーします。結果は正しいですが、完全な再解析は非常に遅くなる可能性があります。現在、これらの入力は次のとおりです。
- `--vars`
- `profiles.yml` の内容（またはその中で使用されている `env_var` の値）
- `dbt_project.yml` の内容（またはその中で使用されている `env_var` の値）
- インストール済みパッケージ
- dbt のバージョン
- 広く使用されている特定のマクロ（例: [builtins](/reference/dbt-jinja-functions/builtins)、オーバーライド、`database`/`schema`/`alias` の `generate_x_name`）

[CI](/docs/deploy/continuous-integration) ジョブ実行をトリガーする場合、部分解析のメリットは新しいプルリクエスト (PR) や新しいブランチには適用されません。ただし、新しい PR またはブランチへの後続のコミットには適用されます。

悪い状態になった場合は、`PARTIAL_PARSE` グローバル設定を false に設定するか、`target/partial_parse.msgpack` を削除する (例: `dbt clean` を実行する) ことで、部分解析を無効にして完全な再解析をトリガーできます。

## 静的パーサー

解析時に、dbt はプロジェクト内のすべてのモデルから `ref()`、`source()`、`config()` の内容を抽出する必要があります。従来、dbt は各モデルファイル内の Jinja をレンダリングすることでこれらの値を抽出していましたが、これは処理速度が遅くなることがありました。そこで、私たちは [`tree-sitter`](https://github.com/tree-sitter/tree-sitter) を活用してモデルファイルを静的に解析します。初期の Jinja2 文法のコードは [こちら](https://github.com/dbt-labs/tree-sitter-jinja2) でご覧いただけます。

静的パーサーはデフォルトで **オン** になっています。これにより、最大 95% のプロジェクトで *ある程度の* 速度向上が期待できます。必要に応じて、[`STATIC_PARSER` グローバル設定](/reference/global-configs/parsing) を使用して無効にすることもできます。

現時点では、静的パーサーはモデル、およびJinjaが3つの特殊マクロ（`ref`、`source`、`config`）に限定されているモデルでのみ動作します。静的パーサーは、完全なJinjaレンダリングよりも少なくとも3倍高速です。dbt Cloudのデータを用いたテストに基づき、現在の文法では実環境のモデルの60%を静的に解析できると考えています。したがって、平均的なプロジェクトでは、40%の高速化が期待できます。model parser.

## 試験的なパーサー

現在は使用されていません。
