---
title: "フラグについて（グローバル設定）"
id: "about-global-configs"
sidebar: "フラグについて（グローバル設定）"
pagination_next: null
---

dbt において、「フラグ」（「グローバル設定」とも呼ばれます）は、dbt がプロジェクトをどのように実行するかを微調整するための設定です。これは、dbt に何を実行するかを指示する [リソース固有の設定](/reference/configs-and-properties) とは異なります。

フラグは、ログの視覚的な出力、特定の警告メッセージをエラーとして扱うかどうか、最初のエラー発生後に「fail fast」するかどうかなどを制御します。フラグはすべての dbt コマンドで使用でき、複数の場所で設定できるため、「グローバル」設定と呼ばれます。

dbt のフラグと dbt のコマンドラインオプションには多くの共通点がありますが、違いもあります。
- 一部のフラグは [`dbt_project.yml`](/reference/dbt_project.yml) でのみ設定でき、特定の呼び出しに対して CLI オプションで上書きすることはできません。
- CLI オプションがすべてのコマンド (「グローバル」) でサポートされているのではなく、特定のコマンドでサポートされている場合、通常は「フラグ」とは見なされません。

### フラグの設定

フラグの設定方法はユースケースに応じて複数あります。
- **[`dbt_project.yml` 内のプロジェクトレベルの `flags`](/reference/global-configs/project-flags):** このプロジェクトを実行するすべてのユーザーに対して、バージョン管理されたデフォルトを定義します。また、[動作の変更](/reference/global-configs/behavior-changes)をオプトインまたはオプトアウトすることで、従来の機能からの移行を管理できます。
- **[環境変数](/reference/global-configs/environment-variable-configs):** 異なるランタイム環境（開発環境、本番環境、継続的インテグレーション）ごとに異なる動作を定義したり、開発環境のユーザーごとに異なる動作（個人の好みに基づいて）を定義したりします。
- **[CLI オプション](/reference/global-configs/command-line-options):** _この呼び出し_ に固有の動作を定義します。すべての dbt コマンドでサポートされています。

最も具体的な設定が「優先」されます。3 つの場所すべてで同じフラグを設定した場合、CLI オプションが優先され、次に環境変数、最後に `dbt_project.yml` の値が優先されます。これらのいずれの場所でもフラグを設定しない場合は、dbt 内で定義されたデフォルト値が使用されます。

ほとんどのフラグは、3 つの場所すべてで設定できます:

```yaml
# dbt_project.yml
flags:
  # set default for running this project -- anywhere, anytime, by anyone
  fail_fast: true
```
```bash
# set this environment variable to 'True' (bash syntax)
export DBT_FAIL_FAST=1
dbt run
```
```bash
dbt run --fail-fast # set to True for this specific invocation
dbt run --no-fail-fast # set to False
```

例外は 2 つのカテゴリに分かれています:
1. **ファイルパスを設定するフラグ:** ランタイム実行に関連するファイルパスのフラグ (例: `--log-path` または `--state`) は、`dbt_project.yml` では設定できません。デフォルトをオーバーライドするには、CLI オプションを渡すか、環境変数 (`DBT_LOG_PATH`、`DBT_STATE`) を設定します。プロジェクト リソースの場所を dbt に指示するフラグ (例: `model-paths`) は、`dbt_project.yml` で設定されますが、`flags` ディクショナリの外側にある最上位キーとして設定されます。これらの構成は完全に静的であり、コマンドや実行環境によって変化することはありません。
2. **オプトイン フラグ:** [動作の変更](/reference/global-configs/behavior-changes) をオプトインまたはオプトアウトするフラグは、`dbt_project.yml` でのみ定義できます。これらはバージョン管理で設定され、プル/マージリクエストによって移行されることを想定しています。これらの値は、呼び出し、環境、またはユーザー間で無期限に異なるべきではありません。

### フラグへのアクセス

Jinja で記述されたカスタムユーザー定義ロジックでは、[`flags` コンテキスト変数](/reference/dbt-jinja-functions/flags) を使用してフラグの値を確認できます。

```yaml
# dbt_project.yml

on-run-start:
  - '{{ log("I will stop at the first sign of trouble", info = true) if flags.FAIL_FAST }}'
```

`flags` の値は呼び出しごとに異なる可能性があるため、dbt が [解析中](/reference/parsing#known-limitations) に解決する構成または依存関係 (`ref` + `source`) への入力として `flags` を使用しないことを強くお勧めします。

## 利用可能なフラグ

| Flag name | Type | Default | Supported in project? | Environment variable | Command line option | Supported in Cloud CLI? |
|-----------|------|---------|-----------------------|----------------------|---------------------|-------------------------|
| [cache_selected_only](/reference/global-configs/cache) | boolean | False | ✅ | `DBT_CACHE_SELECTED_ONLY` | `--cache-selected-only`, `--no-cache-selected-only` | ✅ |
| [debug](/reference/global-configs/logs#debug-level-logging) | boolean | False | ✅ | `DBT_DEBUG` | `--debug`, `--no-debug` | ✅ |
| [defer](/reference/node-selection/defer) | boolean | False | ❌ | `DBT_DEFER` | `--defer`, `--no-defer` | ✅ (enabled by default) |
| [defer_state](/reference/node-selection/defer) | path | None | ❌ | `DBT_DEFER_STATE` | `--defer-state` | ❌ |
| [fail_fast](/reference/global-configs/failing-fast) | boolean | False | ✅ | `DBT_FAIL_FAST` | `--fail-fast`, `-x`, `--no-fail-fast` | ✅ |
| [full_refresh](/reference/resource-configs/full_refresh) | boolean | False | ✅ (as resource config) | `DBT_FULL_REFRESH` | `--full-refresh`, `--no-full-refresh` | ✅ |
| [indirect_selection](/reference/node-selection/test-selection-examples#syntax-examples) | enum | eager | ✅ | `DBT_INDIRECT_SELECTION` | `--indirect-selection` | ❌ |
| [introspect](/reference/commands/compile#introspective-queries) | boolean | True | ❌ | `DBT_INTROSPECT` | `--introspect`, `--no-introspect` | ❌ |
| [log_cache_events](/reference/global-configs/logs#logging-relational-cache-events) | boolean | False | ❌ | `DBT_LOG_CACHE_EVENTS` | `--log-cache-events`, `--no-log-cache-events` | ❌ |
| [log_format_file](/reference/global-configs/logs#log-formatting) | enum | default (text) | ✅ | `DBT_LOG_FORMAT_FILE` | `--log-format-file` | ❌ |
| [log_format](/reference/global-configs/logs#log-formatting) | enum | default (text) | ✅ | `DBT_LOG_FORMAT` | `--log-format` | ❌ |
| [log_level_file](/reference/global-configs/logs#log-level) | enum | debug | ✅ | `DBT_LOG_LEVEL_FILE` | `--log-level-file` | ❌ |
| [log_level](/reference/global-configs/logs#log-level) | enum | info | ✅ | `DBT_LOG_LEVEL` | `--log-level` | ❌ |
| [log_path](/reference/global-configs/logs) | path | None (uses `logs/`) | ❌ | `DBT_LOG_PATH` | `--log-path` | ❌ |
| [partial_parse](/reference/global-configs/parsing#partial-parsing) | boolean | True | ✅ | `DBT_PARTIAL_PARSE` | `--partial-parse`, `--no-partial-parse` | ✅ |
| [populate_cache](/reference/global-configs/cache) | boolean | True | ✅ | `DBT_POPULATE_CACHE` | `--populate-cache`, `--no-populate-cache` | ✅ |
| [print](/reference/global-configs/print-output#suppress-print-messages-in-stdout) | boolean | True | ❌ | `DBT_PRINT` | `--print` | ❌ |
| [printer_width](/reference/global-configs/print-output#printer-width) | int | 80 | ✅ | `DBT_PRINTER_WIDTH` | `--printer-width` | ❌ |
| [profile](/docs/core/connect-data-platform/connection-profiles#about-profiles) | string | None | ✅ (as top-level key) | `DBT_PROFILE`  | `--profile` | ❌ |
| [profiles_dir](/docs/core/connect-data-platform/connection-profiles#about-profiles) | path | None (current dir, then HOME dir) | ❌ | `DBT_PROFILES_DIR` | `--profiles-dir` | ❌ |
| [project_dir](/reference/dbt_project.yml) | path |  | ❌ | `DBT_PROJECT_DIR` | `--project-dir` | ❌ |
| [quiet](/reference/global-configs/logs#suppress-non-error-logs-in-output) | boolean | False | ❌ | `DBT_QUIET` | `--quiet` | ✅ |
| [resource-type](/reference/global-configs/resource-type) (v1.8+) | string | None | ❌ | `DBT_RESOURCE_TYPES` <br></br> `DBT_EXCLUDE_RESOURCE_TYPES` | `--resource-type` <br></br> `--exclude-resource-type` | ✅ |
| [send_anonymous_usage_stats](/reference/global-configs/usage-stats) | boolean | True | ✅ | `DBT_SEND_ANONYMOUS_USAGE_STATS` | `--send-anonymous-usage-stats`, `--no-send-anonymous-usage-stats` | ❌ |
| [source_freshness_run_project_hooks](/reference/global-configs/behavior-changes#source_freshness_run_project_hooks) | boolean | False | ✅ | ❌ | ❌ | ❌ |
| [state](/reference/node-selection/defer) | path | none | ❌ | `DBT_STATE`, `DBT_DEFER_STATE` | `--state`, `--defer-state` | ❌ |
| [static_parser](/reference/global-configs/parsing#static-parser) | boolean | True | ✅ | `DBT_STATIC_PARSER` | `--static-parser`, `--no-static-parser` | ❌ |
| [store_failures](/reference/resource-configs/store_failures) | boolean | False | ✅ (as resource config) | `DBT_STORE_FAILURES` | `--store-failures`, `--no-store-failures` | ✅ |
| [target_path](/reference/global-configs/json-artifacts) | path | None (uses `target/`) | ❌ | `DBT_TARGET_PATH` | `--target-path` | ❌ |
| [target](/docs/core/connect-data-platform/connection-profiles#about-profiles) | string | None | ❌ | `DBT_TARGET` | `--target` | ❌ |
| [use_colors_file](/reference/global-configs/logs#color) | boolean | True | ✅ | `DBT_USE_COLORS_FILE` | `--use-colors-file`, `--no-use-colors-file` | ❌ |
| [use_colors](/reference/global-configs/print-output#print-color) | boolean | True | ✅ | `DBT_USE_COLORS` | `--use-colors`, `--no-use-colors` | ❌ |
| [use_experimental_parser](/reference/global-configs/parsing#experimental-parser) | boolean | False | ✅ | `DBT_USE_EXPERIMENTAL_PARSER` | `--use-experimental-parser`, `--no-use-experimental-parser` | ❌ |
| [version_check](/reference/global-configs/version-compatibility) | boolean | varies | ✅ | `DBT_VERSION_CHECK` | `--version-check`, `--no-version-check` | ❌ |
| [warn_error_options](/reference/global-configs/warnings) | dict | {} | ✅ | `DBT_WARN_ERROR_OPTIONS` | `--warn-error-options` | ✅ |
| [warn_error](/reference/global-configs/warnings) | boolean | False | ✅ | `DBT_WARN_ERROR` | `--warn-error` | ✅ |
| [write_json](/reference/global-configs/json-artifacts) | boolean | True | ✅ | `DBT_WRITE_JSON` | `--write-json`, `--no-write-json` | ✅ |
