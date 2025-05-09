---
title: "動作の変更"
id: "behavior-changes"
sidebar: "動作の変更"
---

import StateModified from '/snippets.ja/_state-modified-compare.md';

ほとんどのフラグは、複数の有効な選択肢を持つ実行時動作を構成するために存在します。適切な選択は、環境、ユーザーの設定、または特定の呼び出しによって異なります。

別のカテゴリのフラグは、既存のプロジェクトに、dbt の新しいリリースで変更される実行時動作への移行期間を提供します。これらのフラグは、次のような方法で、本来であれば矛盾する可能性のあるこれらの目標のバランスをとるのに役立ちます。
- 新規ユーザー/プロジェクトに、より適切で、より合理的で、より一貫性のあるデフォルトの動作を提供します。
- 既存のユーザー/プロジェクトに移行期間を提供します。警告なしに一夜にして変更されることはありません。
- dbt ソフトウェアの保守性を提供します。動作が分岐するたびに、追加のテストと認知的オーバーヘッドが必要になり、将来の開発が遅れます。これらのフラグは、「現在の」動作から「より良い」動作への移行を容易にするために存在するものであり、永久に残るためのものではありません。

これらのフラグは、3 つの開発フェーズを経ます。
1. **導入（デフォルトでは無効）：** dbt は、「古い」動作と「新しい」動作の両方をサポートするためのロジックを追加します。 「新しい」動作はフラグによって制御され、デフォルトでは無効になっているため、古い動作が保持されます。
2. **成熟度（デフォルトで有効）:** フラグのデフォルト値が `false` から `true` に切り替えられ、新しい動作がデフォルトで有効になります。ユーザーはプロジェクトでフラグを `false` に設定することで、「古い」動作を保持し、「新しい」動作をオプトアウトできます。その場合、非推奨の警告が表示される場合があります。
3. **削除（通常は有効）:** フラグを非推奨としてマークした後、そのフラグと、それがサポートしていた「古い」動作を dbt コードベースから削除します。ほとんどのフラグは無期限にサポートすることを目指していますが、永久にサポートすることを約束しているわけではありません。フラグを削除する場合は、事前に十分な通知を行います。

## 動作の変更とは何ですか?

同じ dbt プロジェクト コードと同じ dbt コマンドでも、動作変更前は同じ結果を返しますが、動作変更後は異なる結果を返します。

動作変更の例:
- dbt が以前は発生しなかった検証エラーを発生するようになります。
- dbt が組み込みマクロのシグネチャを変更します。プロジェクトにはそのマクロのカスタム再実装が含まれています。カスタム再実装には受け入れることができない引数が渡されるため、エラーが発生する可能性があります。
- dbt アダプタが、dbt-Jinja コンテキストの `{{ アダプタ }}` オブジェクトで以前は使用可能だったメソッドの名前を変更または削除します。
- dbt は、必須フィールドの削除、既存フィールドの名前またはタイプの変更、または既存フィールドのデフォルト値の削除により、縮小メタデータ アーティファクトに重大な変更を加えます ([README](https://github.com/dbt-labs/dbt-core/blob/37d382c8e768d1e72acd767e0afdcb1f0dc5e9c5/core/dbt/artifacts/README.md#breaking-changes))。
- dbt は、[構造化ログ](/reference/events-logging#structured-logging) からフィールドの 1 つを削除します。

以下は動作の変更ではありません:
- 以前の動作に欠陥があった、望ましくない、または文書化されていないバグを修正しました。
- dbt は、以前は発生していなかった警告を発生するようになりました。
- dbt は、ログ イベント内のわかりやすいメッセージの言語を更新しました。
- dbt は、デフォルトを持つ新しいフィールドを追加するか、デフォルトを持つフィールドを削除することで、契約されたメタデータ成果物に非破壊的な変更を加えます ([README](https://github.com/dbt-labs/dbt-core/blob/37d382c8e768d1e72acd767e0afdcb1f0dc5e9c5/core/dbt/artifacts/README.md#non-breaking-changes))。

変更の大部分は動作変更ではありません。これらの変更の導入にはユーザー側でのアクションは必要ないため、dbt Cloud の継続リリースと dbt Core のパッチリリースに含まれています。

一方、動作変更の移行は、動作変更フラグによって促進され、数か月かけてゆっくりと行われます。フラグは特定の dbt ランタイムバージョンに疎結合されています。フラグを設定することで、ユーザーはこれらの変更をオプトイン（および後でオプトアウト）するかどうかを制御できます。

## 動作変更フラグ

これらのフラグは、`dbt_project.yml` の `flags` ディクショナリに必ず設定する必要があります。これらのフラグはプロジェクトコードに密接に関連した動作を設定するため、バージョン管理で定義し、プルリクエストまたはマージリクエストを通じて変更し、同じテストとピアレビューを受ける必要があります。

次の例は、最新の dbt Cloud および dbt Core バージョンにおける現在のフラグとそのデフォルト値を示しています。特定の動作変更をオプトアウトするには、`dbt_project.yml` でフラグの値を `False` に設定します。オプトアウトした従来の動作に関する警告は、次のいずれかを行うまで引き続き表示されます。

- 問題を解決する（フラグを `True` に切り替える）
- `warn_error_options.silence` フラグを使用して警告を非表示にする

使用可能な動作変更フラグとそのデフォルト値の例を以下に示します:

<File name='dbt_project.yml'>

```yml
flags:
  require_explicit_package_overrides_for_builtin_materializations: False
  require_model_names_without_spaces: False
  source_freshness_run_project_hooks: False
  restrict_direct_pg_catalog_access: False
  require_yaml_configuration_for_mf_time_spines: False
  require_batched_execution_for_custom_microbatch_strategy: False
  require_nested_cumulative_type_params: False
  validate_macro_args: False 
```

</File>

この表は、dbt Cloud の「最新」リリース トラックのどの月と、どのバージョンの dbt Core に動作変更の導入 (デフォルトでは無効) または成熟 (デフォルトでは有効) が含まれているかを示しています。

| Flag                                                            | dbt Cloud "Latest": Intro | dbt Cloud "Latest": Maturity | dbt Core: Intro | dbt Core: Maturity | 
|-----------------------------------------------------------------|------------------|---------------------|-----------------|--------------------|
| [require_explicit_package_overrides_for_builtin_materializations](#package-override-for-built-in-materialization) | 2024.04          | 2024.06             | 1.6.14, 1.7.14  | 1.8.0             |
| [require_resource_names_without_spaces](#no-spaces-in-resource-names)                           | 2024.05          | TBD*                | 1.8.0           | 1.10.0             |
| [source_freshness_run_project_hooks](#project-hooks-with-source-freshness)                              | 2024.03          | TBD*                | 1.8.0           | 1.10.0             |
| [restrict_direct_pg_catalog_access](/reference/global-configs/redshift-changes#the-restrict_direct_pg_catalog_access-flag) [Redshift]   | 2024.09          | TBD*                | dbt-redshift v1.9.0           | 1.9.0             |
| [skip_nodes_if_on_run_start_fails](#failures-in-on-run-start-hooks)                                | 2024.10          | TBD*                | 1.9.0           | TBD*              |
| [state_modified_compare_more_unrendered_values](#source-definitions-for-state)                   | 2024.10          | TBD*                | 1.9.0           | TBD*              |
| [require_yaml_configuration_for_mf_time_spines](#metricflow-time-spine-yaml)                  | 2024.10          | TBD*                | 1.9.0           | TBD*              |
| [require_batched_execution_for_custom_microbatch_strategy](#custom-microbatch-strategy)                  | 2024.11         | TBD*                | 1.9.0           | TBD*              |
| [cumulative_type_params](#cumulative-metrics)         |   2024.11         | TBD*                 | 1.9.0           | TBD*            |
| [validate_macro_args](#macro-argument-validation)         | 2025.03           | TBD*                 | 1.10.0          | TBD*            | 

dbt クラウド成熟度が「TBD」の場合、これらのフラグのデフォルト値が変更される正確な日付はまだ決定されていません。影響を受けるユーザーには、それまでの間、非推奨の警告が表示され、成熟期日前に事前警告を知らせるメールが送信されます。それまでの間、非推奨の警告が表示されている場合は、次のいずれかを実行できます。
- プロジェクトを新しい動作をサポートするように移行し、フラグを「True」に設定して警告を表示しないようにします。
- フラグを「False」に設定します。成熟期日（デフォルト値が変更される日）以降も、警告は引き続き表示され、従来の動作が維持されます。

### on-run-start フックの失敗

このフラグはデフォルトで `False` です。

`skip_nodes_if_on_run_start_fails` フラグを `True` に設定すると、`on-run-start` フックで失敗が発生した場合に、選択したすべてのリソースの実行がスキップされます。

### state:modified のソース定義

:::info

<StateModified features={'/snippets/_state-modified-compare.md'}/>

:::

このフラグはデフォルトで `False` です。

`state_modified_compare_more_unrendered_values` を `True` に設定すると、`state:modified` チェック中の誤検知を削減できます（特に、`prod` と `dev` のようにターゲット環境によって構成が異なる場合）。

このフラグを `True` に設定すると、`state:modified` の比較において、レンダリングされた値ではなくレンダリングされていない値が使用されます。これは、モデルの解析中は `unrendered_config` を、ソースの解析中は `unrendered_database` と `unrendered_schema` の構成を永続化することで実現されます。

###  組み込みマテリアライゼーションのパッケージオーバーライド

`require_explicit_package_overrides_for_builtin_materializations` フラグを `True` に設定すると、この自動オーバーライドが防止されます。

インストール済みパッケージが明示的なオプトインなしに組み込みマテリアライゼーションをオーバーライドする動作は非推奨となりました。このフラグを `True` に設定すると、組み込みマテリアライゼーションの名前と一致するパッケージで定義されたマテリアライゼーションは、検索および解決順序に含まれなくなります。マクロとは異なり、マテリアライゼーションはプロジェクトの `dispatch` 設定で定義された `search_order` を使用しません。

組み込みマテリアライゼーションは、モデルの場合は `'view'`、`'table'`、`'incremental'`、`'materialized_view'` で、モデルの場合は `'test'`、`'unit'`、`'snapshot'`、`'seed'`、`'clone'` です。

ルートプロジェクトで組み込みのマテリアライゼーションを再実装し、パッケージ実装をラップすることで、組み込みのマテリアライゼーションを明示的にオーバーライドし、パッケージで定義されたマテリアライゼーションを優先することができます。

<File name='macros/materialization_view.sql'>

```sql
{% materialization view, snowflake %}
  {{ return(my_installed_package_name.materialization_view_snowflake()) }}
{% endmaterialization %}
```

</File>

将来的には、プロジェクト レベルの [`dispatch` 構成](/reference/project-configs/dispatch-config) を拡張して、組み込みのマテリアライゼーションをオーバーライドするための承認済みパッケージのリストをサポートする可能性があります。

<VersionBlock lastVersion="1.7">

The following flags were introduced in a future version of dbt Core. If you're still using an older version, then you have the legacy behavior by default (when each flag is `False`). 

</VersionBlock>

### リソース名にスペースを使用しないでください

`require_resource_names_without_spaces` フラグは、リソース名にスペースが含まれていないことを強制します。

dbt リソース（モデル、ソースなど）の名前には、文字、数字、アンダースコアを含める必要があります。その他の文字、特にスペースの使用は強く推奨されません。そのため、リソース名でのスペースの使用は非推奨となりました。`require_resource_names_without_spaces` フラグを `True` に設定すると、dbt はリソース名にスペースが含まれていることを検出した場合、非推奨警告ではなく例外を発生させます。

<File name='models/model name with spaces.sql'>

```sql
-- This model file should be renamed to model_name_with_underscores.sql
```

</File>

### ソースフレッシュネスを使用したプロジェクトフック

`source_freshness_run_project_hooks` フラグを `True` に設定すると、`dbt source freshness` コマンドの実行に「プロジェクトフック」（[`on-run-start` / `on-run-end`](/reference/project-configs/on-run-start-on-run-end)）が含まれます。

`source freshness` コマンドの前後に実行したくない特定のプロジェクトフック（[`on-run-start` / `on-run-end`](/reference/project-configs/on-run-start-on-run-end)）がある場合は、それらのフックに条件チェックを追加できます。

<File name='dbt_project.yml'>

```yaml
on-run-start:
  - '{{ ... if flags.WHICH != 'freshness' }}'
```
</File>


### MetricFlow タイムスパイン YAML

`require_yaml_configuration_for_mf_time_spines` フラグはデフォルトで `False` に設定されています。

以前のバージョン（dbt Core 1.8 以前）では、MetricFlow タイムスパインの設定は `metricflow_time_spine.sql` ファイルに保存されていました。

このフラグが `True` に設定されている場合、dbt は引き続き SQL ファイル設定をサポートします。このフラグが `False` に設定されている場合、dbt は SQL ファイルで設定された MetricFlow タイムスパインを検出すると、非推奨の警告を表示します。

MetricFlow YAML ファイルには `time_spine:` フィールドが必要です。詳細については、[MetricFlow timespine](/docs/build/metricflow-time-spine) を参照してください。

### カスタムマイクロバッチ戦略

`require_batched_execution_for_custom_microbatch_strategy` フラグはデフォルトで `False` に設定されており、プロジェクトにカスタムマイクロバッチマクロが既に存在する場合にのみ有効です。カスタムマイクロバッチマクロがない場合は、このフラグを設定する必要はありません。dbt は [マイクロバッチ戦略](/docs/build/incremental-microbatch#how-microbatch-compares-to-other-incremental-strategies) を使用するすべてのモデルに対してマイクロバッチ処理を自動的に処理します。

プロジェクトにカスタムマイクロバッチマクロが設定されている場合は、このフラグを `True` に設定してください。このフラグが `True` に設定されている場合、dbt はカスタムマイクロバッチ戦略をバッチで実行します。

カスタムマイクロバッチマクロがあり、このフラグが `False` のままになっている場合、dbt は非推奨の警告を発行します。

以前は、既存のカスタム増分戦略との意図しない相互作用を防ぐため、ユーザーは`DBT_EXPERIMENTAL_MICROBATCH`環境変数を`True`に設定する必要がありました。しかし、`DBT_EXPERMINENTAL_MICROBATCH`の設定がランタイム機能に影響を与えなくなったため、この設定は不要になりました。

### 累積メトリクス

[累積型メトリクス](/docs/build/cumulative#parameters)は、[dbt Cloud "最新" リリーストラック](/docs/dbt-versions/cloud-release-tracks)、dbt Core v1.9以降では、`cumulative_type_params`フィールドの下にネストされます。現在、累積メトリクスが不適切にネストされている場合、dbtはユーザーに警告を表示します。新しい形式を適用するには（警告ではなくエラーになります）、`require_nested_cumulative_type_params`を`True`に設定してください。

例として、v1.9より前の構文で構成された次のメトリクスを使用します:

```yaml

    type: cumulative
    type_params:
      measure: order_count
      window: 7 days

```

Core v1.9 または [dbt Cloud "最新" リリース トラック](/docs/dbt-versions/cloud-release-tracks) でその構文を使用して `dbt parse` を実行すると、次のような警告が表示されます:

```bash

15:36:22  [WARNING]: Cumulative fields `type_params.window` and
`type_params.grain_to_date` has been moved and will soon be deprecated. Please
nest those values under `type_params.cumulative_type_params.window` and
`type_params.cumulative_type_params.grain_to_date`. See documentation on
behavior changes:
https://docs.getdbt.com/reference/global-configs/behavior-changes.

```

`require_nested_cumulative_type_params` を `True` に設定して `dbt parse` を再実行すると、次のようなエラーが表示されます:

```bash

21:39:18  Cumulative fields `type_params.window` and `type_params.grain_to_date` should be nested under `type_params.cumulative_type_params.window` and `type_params.cumulative_type_params.grain_to_date`. Invalid metrics: orders_last_7_days. See documentation on behavior changes: https://docs.getdbt.com/reference/global-configs/behavior-changes.

```

メトリックが更新されると、期待どおりに動作するようになります:

```yaml

    type: cumulative
    type_params:
      measure:
        name: order_count
      cumulative_type_params:
        window: 7 days

```

### マクロ引数の検証

dbt は、`validate_macro_args` フラグを使用したマクロ引数の検証（オプション）をサポートしています。デフォルトでは、`validate_macro_args` フラグは `False` に設定されており、これは dbt がドキュメント化されたマクロ引数の名前または型を検証しないことを意味します。

これまで、dbt は YAML 形式のマクロ引数の [`type`](/reference/resource-properties/argument-type) フィールドに標準的な語彙を強制していませんでした。そのため、`type` フィールドはドキュメント化のみに使用され、dbt は以下の点を確認していませんでした。
- 引数名がマクロ内の引数名と一致している
- 引数の型が有効であるか、またはマクロの Jinja 定義と一致している

ドキュメント化されたマクロの例を以下に示します:

<File name='macros/filename.yml'>

```yaml
version: 2

macros:
  - name: <macro name>
    arguments:
      - name: <arg name>
        type: <string>
```
</File>

`validate_macro_args` フラグを `True` に設定すると、dbt は以下の処理を行います。
- YAML 内のすべての引数名がマクロ定義の引数名と一致していることを確認します。
- 名前または型が一致しない場合は警告を発します。
- `types` 値が、次のセクションで説明するサポートされている形式に従っていることを確認します。
- このフラグを使用し、YAML に引数が記述されていない場合、dbt はマクロから引数を推測し、[`manifest.json` ファイル](/reference/artifacts/manifest-json) に含めます。

#### サポートされている型

dbt は、マクロ引数として以下の型をサポートしています。

- `string` または `str`
- `boolean` または `bool`
- `integer` または `int`
- `float`
- `any`
- `list[<Type>]`（例：`list[string]`）
- `dict[<Type>, <Type>]`（例：`dict[str, list[int]]`）
- `optional[<Type>]`（例：`optional[integer]`）
- [`relation`](/reference/dbt-classes#relation)
- [`column`](/reference/dbt-classes#column)

これらの型は Python 風のスタイルに従っていますが、ドキュメント作成と検証のみに使用されます。Python の型ではありません。
