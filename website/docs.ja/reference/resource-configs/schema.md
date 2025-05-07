---
sidebar_label: "schema"
resource_types: [models, seeds, tests]
description: "dbt がデータ プラットフォームにリソースを作成するときに、デフォルトのスキーマをオーバーライドします。"
datatype: string
---

<Tabs>
<TabItem value="model" label="Model">

`dbt_project.yml` ファイルまたは [config ブロック](/reference/resource-configs/schema#models) で、モデルグループに [カスタムスキーマ](/docs/build/custom-schemas#understanding-custom-schemas) を指定します。

例えば、マーケティング関連のモデルグループがあり、それらを `marketing` という別のスキーマに配置する場合は、次のように設定します。

<File name='dbt_project.yml'>

```yml
models:
  your_project:
    marketing: #  Grouping or folder for set of models
      +schema: marketing
```
</File>


これにより、これらのモデルに対して生成されたリレーションは `marketing` スキーマに配置され、完全なリレーション名は `analytics.target_schema_marketing.model_name` となります。これは、リレーションのスキーマが `{{ target.schema }}_{{ schema }}` であるためです。[定義](#definition) セクションで、この点について詳しく説明されています。

</TabItem>

<TabItem value="seeds" label="Seeds">

`dbt_project.yml` ファイルで [カスタムスキーマ](/docs/build/custom-schemas#understanding-custom-schemas) を設定します。

例えば、`mappings` という別のスキーマに配置するシードがある場合は、次のように設定できます。

<File name='dbt_project.yml'>

```yml
seeds:
  your_project:
    product_mappings:
      +schema: mappings
```

これにより、生成されたリレーションは `mappings` スキーマに配置されるため、完全なリレーション名は `analytics.mappings.seed_name` になります。

</File>
</TabItem>

<TabItem value="snapshots" label="Snapshots">

<VersionBlock lastVersion="1.8">

Available in dbt Core v1.9 and higher. Select v1.9 or newer from the version dropdown to view the configs. Try it now in the [dbt Cloud "Latest" release track](/docs/dbt-versions/cloud-release-tracks).

</VersionBlock>

<VersionBlock firstVersion="1.9">

`dbt_project.yml` ファイルまたは YAML ファイルで、スナップショットの [カスタム スキーマ](/docs/build/custom-schemas#understanding-custom-schemas) を指定します。

たとえば、ターゲット スキーマ以外のスキーマに読み込むスナップショットがある場合は、次のように設定します。

`dbt_project.yml` ファイルで、次の設定を行います:

<File name='dbt_project.yml'>

```yml
snapshots:
  your_project:
    your_snapshot:
      +schema: snapshots
```
</File>

`snapshots/snapshot_name.yml` ファイル内:

<File name='snapshots/snapshot_name.yml'>

```yaml
version: 2

snapshots:
  - name: snapshot_name
    [config](/reference/resource-properties/config):
      schema: snapshots
```

</File>

この結果、生成されたリレーションは `snapshots` スキーマに配置されるため、完全なリレーション名はデフォルトのターゲット スキーマではなく `analytics.snapshots.your_snapshot` になります。

</VersionBlock>

</TabItem>

<TabItem value="saved-queries" label="Saved queries">

`dbt_project.yml` または YAML ファイルで、[保存されたクエリ](/docs/build/custom-schemas#understanding-custom-schemas) の [カスタム スキーマ](/docs/build/custom-schemas#understanding-custom-schemas) を指定します。

<File name='dbt_project.yml'>
```yml
saved-queries:
  +schema: metrics
```
</File>

これにより、保存されたクエリが `metrics` スキーマに保存されることになります。

</TabItem>

<TabItem value="tests" label="Test">

`dbt_project.yml` ファイルでテスト結果を保存するための [カスタムスキーマ](/docs/build/custom-schemas#understanding-custom-schemas) をカスタマイズします。

たとえば、テスト結果を特定のスキーマに保存するには、次のように設定します:

<File name='dbt_project.yml'>

```yml
tests:
  +store_failures: true
  +schema: test_results
```

これにより、テスト結果は `test_results` スキーマに保存されます。
</File>
</TabItem>
</Tabs>

詳細な例については、[使用方法](#usage) を参照してください。

## 定義

オプションで、[モデル](/docs/build/sql-models)、[シード](/docs/build/seeds)、[スナップショット](/docs/build/snapshots)、[保存済みクエリ](/docs/build/saved-queries)、または[テスト](/docs/build/data-tests)のカスタムスキーマを指定します。

dbt Cloud v1.8 以前のバージョンをご利用の場合は、[`target_schema` 設定](/reference/resource-configs/target_schema) を使用してスナップショットのカスタムスキーマを指定します。

dbt がデータベースにリレーション (<Term id="table" />/<Term id="view" />) を作成する場合、`{{ database }}.{{ schema }}.{{ identifier }}` という形式で作成されます。例: `analytics.finance.payments`

dbt の標準的な動作は次のとおりです。
* カスタムスキーマが指定されていない場合、リレーションのスキーマはターゲットスキーマ (`{{ target.schema }}`) になります。
* カスタムスキーマが指定されている場合、デフォルトでは、リレーションのスキーマは `{{ target.schema }}_{{ schema }}` になります。

dbt がリレーションの `schema` を生成する方法を変更する方法の詳細については、[カスタムスキーマの使用](/docs/build/custom-schemas) を参照してください。

## Usage

### Models

`dbt_project.yml` ファイルからモデルのグループを構成します。

<File name='dbt_project.yml'>

```yml
models:
  jaffle_shop: # the name of a project
    marketing:
      +schema: marketing
```

</File>

構成ブロックを使用して個々のモデルを構成します:

<File name='models/my_model.sql'>

```sql
{{ config(
    schema='marketing'
) }}
```

</File>

### Seeds
<File name='dbt_project.yml'>

```yml
seeds:
  +schema: mappings
```

</File>

### テスト

[失敗を保存するように構成された](/reference/resource-configs/store_failures)テストの結果を保存するスキーマの名前をカスタマイズします。
生成されるスキーマは ```{{ profile.schema }}_{{ tests.schema }}``` で、デフォルトのサフィックスは「dbt_test__audit」です。
同じプロファイルスキーマを使用するには、「+schema: null」を設定します。

<File name='dbt_project.yml'>

```yml
tests:
  +store_failures: true
  +schema: _sad_test_failures  # Will write tables to my_database.my_schema__sad_test_failures
```

</File>

作業に必要なスキーマを作成またはアクセスする権限があることを確認してください。必要なスキーマに適切な権限が付与されていることを確認するには、それぞれのデータプラットフォーム環境でSQL文を実行してください。例えば、Redshiftを使用している場合は、以下のコマンドを実行します（正確な権限クエリはデータプラットフォームによって異なる場合があります）。

```sql
create schema if not exists dev_username_dbt_test__audit authorization username;
```

_`dev_username` を実際の開発スキーマ名に、`username` を権限を付与する適切なユーザーに置き換えてください。_

このコマンドは、`store_failures` 構成でよく使用される `dbt_test__audit` スキーマの作成とアクセスに必要な権限を付与します。

## ウェアハウス固有の情報
* BigQuery: `dataset` と `schema` は互換性があります
