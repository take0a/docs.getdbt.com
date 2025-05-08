---
title: "Node selector メソッド"
sidebar: "Node selector methods"
---

Selector メソッドは、`method:value` という構文を使用して、共通のプロパティを持つすべてのリソースを返します。メソッドを明示的に指定することをお勧めしますが、省略することも可能です（デフォルト値は `path`、`file`、`fqn` のいずれかになります）。

<Expandable alt_header="--select と --selector の違い">

`--select` 引数と `--selector` 引数は似ていますが、実際には異なります。違いを理解するには、[`--select` と `--selector` の違い](/reference/node-selection/yaml-selectors#difference-between---select-and---selector) を参照してください。

</Expandable>

以下のメソッドの多くは Unix スタイルのワイルドカードをサポートしています。

| Wildcard | Description                                               |
| -------- | --------------------------------------------------------- |
| \*       | 任意の数の任意の文字（0文字を含む）に一致します    |
| ?        | 任意の1文字に一致する                              |
| [abc]    | 括弧内の1文字に一致する                |
| [a-z]    | 括弧内の範囲の1文字に一致する |

例えば：
```
dbt list --select "*.folder_name.*"
dbt list --select "package:*_source"
```

### access

`access` メソッドは、[access](/reference/resource-configs/access) プロパティに基づいてモデルを選択します。

```bash
dbt list --select "access:public"      # list all public models
dbt list --select "access:private"       # list all private models
dbt list --select "access:protected"       # list all protected models
```

### config

`config` メソッドは、指定された [ノード構成](/reference/configs-and-properties) に一致するモデルを選択するために使用されます。



  ```bash
dbt run --select "config.materialized:incremental"    # run all models that are materialized incrementally
dbt run --select "config.schema:audit"              # run all models that are created in the `audit` schema
dbt run --select "config.cluster_by:geo_country"      # run all models clustered by `geo_country`
```

ほとんどの設定値は文字列ですが、`config` メソッドを使用して、ブール値の設定、辞書のキー、リスト内の値を一致させることもできます。

例えば、次の設定を持つモデルがあるとします:

```bash
{{ config(
  materialized = 'incremental',
  unique_key = ['column_a', 'column_b'],
  grants = {'select': ['reporter', 'analysts']},
  meta = {"contains_pii": true},
  transient = true
) }}

select ...
```

次のいずれかを使用して選択できます:

```bash
dbt ls -s config.materialized:incremental
dbt ls -s config.unique_key:column_a
dbt ls -s config.grants.select:reporter
dbt ls -s config.meta.contains_pii:true
dbt ls -s config.transient:true
```


### exposure

`exposure`メソッドは、指定された[exposure](/docs/build/exposures)の親リソースを選択するために使用されます。`+`演算子と組み合わせて使用​​してください。


  ```bash
dbt run --select "+exposure:weekly_kpis"                # run all models that feed into the weekly_kpis exposure
dbt test --select "+exposure:*"                         # test all resources upstream of all exposures
dbt ls --select "+exposure:*" --resource-type source    # list all source tables upstream of all exposures
```

### file

`file` メソッドを使用すると、ファイル拡張子 (`.sql`) を含むファイル名でモデルを選択できます。

```bash
# These are equivalent
dbt run --select "file:some_model.sql"
dbt run --select "some_model.sql"
dbt run --select "some_model"
```

### fqn

`fqn` メソッドは、dbt グラフ内の「完全修飾名」（FQN）に基づいてノードを選択するために使用されます。[`dbt list`](/reference/commands/list) のデフォルトの出力は、FQN のリストです。デフォルトの FQN 形式は、プロジェクト名、パス内のサブディレクトリ、およびファイル名（拡張子なし）をピリオドで区切ったもので構成されます。

```bash
dbt run --select "fqn:some_model"
dbt run --select "fqn:your_project.some_model"
dbt run --select "fqn:some_package.some_other_model"
dbt run --select "fqn:some_path.some_model"
dbt run --select "fqn:your_project.some_path.some_model"
```


### group

`group` メソッドは、[グループ](/reference/resource-configs/group) 内で定義されたモデルを選択するために使用されます。


```bash
dbt run --select "group:finance" # run all models that belong to the finance group.
```

### metric

`metric` メソッドは、指定された [metric](/docs/build/build-metrics-intro) の親リソースを選択するために使用されます。`+` 演算子と組み合わせて使用​​してください。

```bash
dbt build --select "+metric:weekly_active_users"       # build all resources upstream of weekly_active_users metric
dbt ls    --select "+metric:*" --resource-type source  # list all source tables upstream of all metrics
```

### package

`package` メソッドは、ルートプロジェクト内またはインストールされた dbt パッケージ内で定義されたモデルを選択するために使用されます。`package:` プレフィックスは明示的には必須ではありませんが、セレクターを明確にするために使用できます。


  ```bash
  # These three selectors are equivalent
  dbt run --select "package:snowplow"
  dbt run --select "snowplow"
  dbt run --select "snowplow.*"
```

現在のプロジェクトからノードを選択するには、`this` パッケージを使用します。例では、`snowplow` プロジェクトから `dbt run --select "package:this"` を実行すると、他の 3 つのセレクターとまったく同じモデルセットが実行されます。

`this` は常に現在のプロジェクトを参照するため、`package:this` を使用すると、作業中のプロジェクトからのみモデルが選択されます。

### path

`path` メソッドは、特定のパスまたはその配下で定義されたモデル/ソースを選択するために使用されます。
モデル定義は SQL/Python ファイル（YAML ではありません）で、ソース定義は YAML ファイルで行われます。
`path` プレフィックスは明示的に必須ではありませんが、セレクターを明確にするために使用できます。


  ```bash
  # These two selectors are equivalent
  dbt run --select "path:models/staging/github"
  dbt run --select "models/staging/github"

  # These two selectors are equivalent
  dbt run --select "path:models/staging/github/stg_issues.sql"
  dbt run --select "models/staging/github/stg_issues.sql"
  ```

### resource_type

特定のタイプのノード（`model`、`test`、`exposure` など）を選択するには、`resource_type` メソッドを使用します。これは、[`dbt ls` コマンド](/reference/commands/list) で使用される `--resource-type` フラグに似ています。

  ```bash
dbt build --select "resource_type:exposure"    # build all resources upstream of exposures
dbt list --select "resource_type:test"         # list all tests in your project
dbt list --select "resource_type:source"       # list all sources in your project
```

### result

`result` メソッドは前述の `state` メソッドと関連しており、前回の実行結果のステータスに基づいてリソースを選択するために使用できます。結果セレクターが操作する結果を作成するには、dbt コマンド [`run`、`test`、`build`、`seed`] のいずれかを実行する必要があります。`result` セレクターは `+` 演算子と組み合わせて使用​​できます。

```bash
dbt run --select "result:error" --state path/to/artifacts # run all models that generated errors on the prior invocation of dbt run
dbt test --select "result:fail" --state path/to/artifacts # run all tests that failed on the prior invocation of dbt test
dbt build --select "1+result:fail" --state path/to/artifacts # run all the models associated with failed tests from the prior invocation of dbt build
dbt seed --select "result:error" --state path/to/artifacts # run all seeds that generated errors on the prior invocation of dbt seed.
```

### saved_query

`saved_query` メソッドは [保存されたクエリ](/docs/build/saved-queries) を選択します。

```bash
dbt list --select "saved_query:*"                    # list all saved queries 
dbt list --select "+saved_query:orders_saved_query"  # list your saved query named "orders_saved_query" and all upstream resources
```

### semantic_model

`semantic_model` メソッドは [セマンティック モデル](/docs/build/semantic-models) を選択します。

```bash
dbt list --select "semantic_model:*"        # list all semantic models 
dbt list --select "+semantic_model:orders"  # list your semantic model named "orders" and all upstream resources
```

### source

`source` メソッドは、指定された [source](/docs/build/sources#using-sources) からモデルを選択するために使用されます。`+` 演算子と組み合わせて使用​​してください。


  ```bash
dbt run --select "source:snowplow+"    # run all models that select from Snowplow sources
```

### source_status
  
ジョブ状態のもう 1 つの要素は、以前の dbt 呼び出しの `source_status` です。たとえば、`dbt source freshness` を実行すると、dbt は `sources.json` アーティファクトを作成します。このアーティファクトには、dbt ソースの実行時間と `max_loaded_at` 日付が含まれます。`sources.json` の詳細については、['sources'](/reference/artifacts/sources-json) ページをご覧ください。

以下の dbt コマンドは、`sources.json` アーティファクトを生成します。その結果は、以降の dbt 呼び出しで参照できます。
- `dbt source freshness`

上記のいずれかのコマンドを実行した後、次のように後続のコマンドにセレクターを追加することで、ソースの鮮度の結果を参照できます。


```bash
# You can also set the DBT_STATE environment variable instead of the --state flag.
dbt source freshness # must be run again to compare current to previous state
dbt build --select "source_status:fresher+" --state path/to/prod/artifacts
```

### state

**注** [状態ベースの選択](/reference/node-selection/state-selection)は強力かつ複雑な機能です。状態比較については、[既知の注意事項と制限事項](/reference/node-selection/state-comparison-caveats)をご覧ください。

`state`メソッドは、同じプロジェクトの以前のバージョン（[マニフェスト](/reference/artifacts/manifest-json)で表されます）と比較してノードを選択するために使用されます。比較マニフェストのファイルパスは、`--state`フラグまたは`DBT_STATE`環境変数で指定する必要があります。

`state:new`: 比較マニフェスト内に同じ`unique_id`を持つノードが存在しません。

`state:modified`: すべての新規ノードと、既存のノードへの変更。

  ```bash
dbt test --select "state:new" --state path/to/artifacts      # run all tests on new models + and new tests on old models
dbt run --select "state:modified" --state path/to/artifacts  # run all models that have been modified
dbt ls --select "state:modified" --state path/to/artifacts   # list all modified nodes (not just models)
  ```

状態の比較は複雑で、プロジェクトごとに異なるため、dbt は完全な `modified` 基準のサブセットを含むサブセレクタをサポートしています。
- `state:modified.body`: ノード本体の変更 (例: モデル SQL、シード値)
- `state:modified.configs`: `database`/`schema`/`alias` を除くすべてのノード構成の変更
- `state:modified.relation`: `database`/`schema`/`alias` (このノードのデータベース表現) の変更 (`target` 値や `generate_x_name` マクロとは無関係)
- `state:modified.persisted_descriptions`: リレーションレベルまたは列レベルの `description` の変更 (各レベルで `persist_docs` が有効になっている場合のみ)
- `state:modified.macros`: アップストリームマクロの変更 (直接呼び出されたか、 (別のマクロによって間接的に)
- `state:modified.contract`: モデルの [コントラクト](/reference/resource-configs/contract) の変更。現在、`columns` の `name` と `data_type` が含まれます。既存の列の型を削除または変更すると、互換性を破る変更とみなされ、エラーが発生します。

`state:modified` には、上記のすべての基準に加えて、ソースの `freshness` や `quoting` ルール、エクスポージャーの `maturity` プロパティの変更など、リソース固有の追加基準も含まれることに注意してください。 ([ソース](https://github.com/dbt-labs/dbt-core/blob/9e796671dd55d4781284d36c035d1db19641cd80/core/dbt/contracts/graph/parsed.py#L660-L681)、[エクスポージャー](https://github.com/dbt-labs/dbt-core/blob/9e796671dd55d4781284d36c035d1db19641cd80/core/dbt/contracts/graph/parsed.py#L768-L783)、および[実行可能ファイル]を比較する際に使用されるチェックの完全なセットについては、ソースコードを参照してください。ノード](https://github.com/dbt-labs/dbt-core/blob/9e796671dd55d4781284d36c035d1db19641cd80/core/dbt/contracts/graph/parsed.py#L319-L330)

`state:new` および `state:modified` を補完し、これらの機能の逆を表す `state` セレクターが 2 つあります。
- `state:old` &mdash; 比較マニフェストに同じ `unique_id` を持つノードが存在する
- `state:unmodified` &mdash; 変更のない既存のすべてのノード

これらのセレクターを使用すると、変更されていないノードを除外することで実行時間を短縮できます。現時点ではサブセレクターは利用できませんが、ユースケースの進化に伴い変更される可能性があります。

#### `state:modified` ノードと参照への影響

`state:modified` は、追加された新しいノード、既存のノードへの変更、および以下の変更を識別します。

- [access](/reference/resource-configs/access) 権限
- [`deprecation_date` ](/reference/resource-properties/deprecation_date)
- [`latest_version` ](/reference/resource-properties/latest_version)

ノードがグループを変更すると、下流の参照が壊れ、ビルドが失敗する可能性があります。

`group` は構成であり、構成は通常 `state:modified` の検出に含まれるため、参照されているすべての場所でグループ名を変更すると、それらのノードは「変更済み」としてフラグ付けされます。

部分解析が有効になっているかどうかに応じて、CI ワークフローの一部として破損を検出できます。

- 参照されているすべての場所でグループ名を変更し、部分解析が有効になっている場合、dbt は変更されたモデルのみを再解析することがあります。
- 部分解析を有効にせずにすべての参照でグループ名を更新した場合、dbt はすべてのモデルを再解析し、無効な下流参照を特定します。

グループ名を変更し、かつ `dbt build --select state:modified` によって実行対象として何かが選択されると、「何もする必要はありません」というエラーが発生する可能性があります。このエラーは、CI ジョブが `state:modified+`（下流を含む）を選択している限り、実行時に検出されます。

参照の使用方法や解決方法には、次のような要因が影響する可能性があります。

- アクセスの変更：権限またはアクセスルールが変更されると、一部の参照が機能しなくなる可能性があります。
- `deprecation_date` の変更：参照またはモデルバージョンが非推奨としてマークされている場合、参照の処理方法に影響する新しい警告が表示されることがあります。
- `latest_version` の変更: 特定のバージョンへの関連付けがない場合、参照またはモデルは最新バージョンを指します。
  - 新しいバージョンがリリースされた場合、参照は自動的に新しいバージョンに解決されるため、それに依存するシステムの動作や出力が変更される場合があります。

#### `manifest.json` を上書きします

import Overwritesthemanifest from '/snippets.ja/_overwrites-the-manifest.md';

<Overwritesthemanifest />

#### おすすめ

import Recommendationoverwritesthemanifest from '/snippets.ja/_recommendation-overwriting-manifest.md'; 

<Recommendationoverwritesthemanifest />

### tag

`tag:` メソッドは、指定された [タグ](/reference/resource-configs/tags) に一致するモデルを選択するために使用されます。


  ```bash
dbt run --select "tag:nightly"    # run all models with the `nightly` tag
```

### test_name

`test_name` メソッドは、そのテストを定義するジェネリックテストの名前に基づいてテストを選択するために使用されます。ジェネリックテストの定義方法の詳細については、[テスト](/docs/build/data-tests) をご覧ください。


  ```bash
dbt test --select "test_name:unique"            # run all instances of the `unique` test
dbt test --select "test_name:equality"          # run all instances of the `dbt_utils.equality` test
dbt test --select "test_name:range_min_max"     # run all instances of a custom schema test defined in the local project, `range_min_max`
```

### The test_type

<VersionBlock lastVersion="1.7">

`test_type` メソッドは、テストのタイプ (`singular` または `generic`) に基づいてテストを選択するために使用されます:

```bash
dbt test --select "test_type:generic"        # run all generic tests
dbt test --select "test_type:singular"       # run all singular tests
```

</VersionBlock>

<VersionBlock firstVersion="1.8">

`test_type` メソッドは、テストの種類に基づいてテストを選択するために使用されます。

- [ユニットテスト](/docs/build/unit-tests)
- [データテスト](/docs/build/data-tests):
  - [Singular](/docs/build/data-tests#singular-data-tests)
  - [Generic](/docs/build/data-tests#generic-data-tests)


```bash
dbt test --select "test_type:unit"           # run all unit tests
dbt test --select "test_type:data"           # run all data tests
dbt test --select "test_type:generic"        # run all generic data tests
dbt test --select "test_type:singular"       # run all singular data tests
```

</VersionBlock>

### unit_test

<VersionBlock lastVersion="1.7">
Supported in v1.8 or newer.
</VersionBlock>
<VersionBlock firstVersion="1.8">

`unit_test` メソッドは [ユニット テスト](/docs/build/unit-tests) を選択します。

```bash
dbt list --select "unit_test:*"                        # list all unit tests 
dbt list --select "+unit_test:orders_with_zero_items"  # list your unit test named "orders_with_zero_items" and all upstream resources
```

</VersionBlock>

### version

`version` メソッドは、[バージョン識別子](/reference/resource-properties/versions) と [最新バージョン](/reference/resource-properties/latest_version) に基づいて [バージョン管理されたモデル](/docs/collaborate/govern/model-versions) を選択します。

```bash
dbt list --select "version:latest"      # only 'latest' versions
dbt list --select "version:prerelease"  # versions newer than the 'latest' version
dbt list --select "version:old"         # versions older than the 'latest' version

dbt list --select "version:none"        # models that are *not* versioned
```
