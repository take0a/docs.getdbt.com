---
description: "dbt_project.yml ファイルを構成するためのリファレンス ガイド。"
intro_text: "dbt_project.yml ファイルは、すべての dbt プロジェクトに必須のファイルです。このファイルには、dbt にプロジェクトの操作方法を指示する重要な情報が含まれています。"
---

すべての [dbt プロジェクト](/docs/build/projects) には `dbt_project.yml` ファイルが必要です。これは、dbt がディレクトリが dbt プロジェクトであることを認識するためのものです。また、このファイルには、dbt にプロジェクトの操作方法を指示する重要な情報も含まれています。動作は以下のとおりです。

- dbt はいくつかの場所で [YAML](https://yaml.org/) を使用します。YAML を初めて使用する場合は、配列、辞書、文字列の表現方法を学ぶことをお勧めします。

- デフォルトでは、dbt は現在の作業ディレクトリとその親ディレクトリで `dbt_project.yml` を検索しますが、`--project-dir` フラグまたは `DBT_PROJECT_DIR` 環境変数を使用して別のディレクトリを指定することもできます。

- `dbt_project.yml` ファイルで、`dbt-cloud` 構成ファイルの `project-id` を使用して、dbt Cloud プロジェクト ID を指定します。 dbt Cloud プロジェクト URL でプロジェクト ID を見つけます。たとえば、`https://YOUR_ACCESS_URL/11/projects/123456` の場合、プロジェクト ID は `123456` です。

- `dbt_project.yml` ファイルでは、設定ファイル（[マクロ](/reference/macro-properties) など）以外の「プロパティ」を設定することはできません。これはすべての種類のリソースに適用されます。詳細については、[設定ファイルとプロパティ](/reference/configs-and-properties) を参照してください。

## 例

次の例は、`dbt_project.yml` ファイルで利用可能なすべての設定のリストです:

<File name='dbt_project.yml'>

```yml
[name](/reference/project-configs/name): string

[config-version](/reference/project-configs/config-version): 2
[version](/reference/project-configs/version): version

[profile](/reference/project-configs/profile): profilename

[model-paths](/reference/project-configs/model-paths): [directorypath]
[seed-paths](/reference/project-configs/seed-paths): [directorypath]
[test-paths](/reference/project-configs/test-paths): [directorypath]
[analysis-paths](/reference/project-configs/analysis-paths): [directorypath]
[macro-paths](/reference/project-configs/macro-paths): [directorypath]
[snapshot-paths](/reference/project-configs/snapshot-paths): [directorypath]
[docs-paths](/reference/project-configs/docs-paths): [directorypath]
[asset-paths](/reference/project-configs/asset-paths): [directorypath]

[packages-install-path](/reference/project-configs/packages-install-path): directorypath

[clean-targets](/reference/project-configs/clean-targets): [directorypath]

[query-comment](/reference/project-configs/query-comment): string

[require-dbt-version](/reference/project-configs/require-dbt-version): version-range | [version-range]

[flags](/reference/global-configs/project-flags):
  [<global-configs>](/reference/global-configs/project-flags)

[dbt-cloud](/docs/cloud/cloud-cli-installation):
  [project-id](/docs/cloud/configure-cloud-cli#configure-the-dbt-cloud-cli): project_id # Required
  [defer-env-id](/docs/cloud/about-cloud-develop-defer#defer-in-dbt-cloud-cli): environment_id # Optional

[exposures](/docs/build/exposures):
  +[enabled](/reference/resource-configs/enabled): true | false

[quoting](/reference/project-configs/quoting):
  database: true | false
  schema: true | false
  identifier: true | false

metrics:
  [<metric-configs>](/docs/build/metrics-overview)

models:
  [<model-configs>](/reference/model-configs)

seeds:
  [<seed-configs>](/reference/seed-configs)

semantic-models:
  [<semantic-model-configs>](/docs/build/semantic-models)

saved-queries:
  [<saved-queries-configs>](/docs/build/saved-queries)

snapshots:
  [<snapshot-configs>](/reference/snapshot-configs)

sources:
  [<source-configs>](source-configs)
  
tests:
  [<test-configs>](/reference/data-test-configs)

vars:
  [<variables>](/docs/build/project-variables)

[on-run-start](/reference/project-configs/on-run-start-on-run-end): sql-statement | [sql-statement]
[on-run-end](/reference/project-configs/on-run-start-on-run-end): sql-statement | [sql-statement]

[dispatch](/reference/project-configs/dispatch-config):
  - macro_namespace: packagename
    search_order: [packagename]

[restrict-access](/docs/collaborate/govern/model-access): true | false

```

</File>

## `+` プレフィックス

import PlusPrefix from '/snippets.ja/_plus-prefix.md';

<PlusPrefix />

## 命名規則

dbt が適切に処理できるように、`dbt_project.yml` ファイル内の設定項目は正しい YAML 命名規則に従うことが重要です。これは、複数の単語を含むリソースタイプの場合に特に重要です。

- `dbt_project.yml` ファイルで複数の単語を含むリソースタイプを設定する場合は、ダッシュ (`-`) を使用します。[保存済みクエリ](/docs/build/saved-queries#configure-saved-query) の例を以下に示します。

    <File name="dbt_project.yml">

    ```yml
    saved-queries:  # Use dashes for resource types in the dbt_project.yml file.
      my_saved_query:
        +cache:
          enabled: true
    ```
    </File>

- `dbt_project.yml` ファイル以外の YAML ファイルで、複数の単語を含むリソースタイプを設定する場合は、アンダースコア (`_`) を使用します。例えば、`semantic_models.yml` ファイルに保存されている同じクエリリソースは次のようになります:

    <File name="models/semantic_models.yml">

    ```yml
    saved_queries:  # Use underscores everywhere outside the dbt_project.yml file.
      - name: saved_query_name
        ... # Rest of the saved queries configuration.
        config:
          cache:
            enabled: true
    ```
    </File>
