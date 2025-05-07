---
sidebar_label: "database"
resource_types: [models, seeds, tests]
datatype: string
description: "dbt がデータ プラットフォームにリソースを作成するときに、デフォルトのデータベースをオーバーライドします。"
---

<Tabs>
<TabItem value="model" label="Model">

`dbt_project.yml` ファイルで、モデルのカスタムデータベースを指定します。

例えば、ターゲットデータベース以外のデータベースにロードしたいモデルがある場合は、次のように設定します:

<File name='dbt_project.yml'>

```yml
models:
  your_project:
    sales_metrics:
      +database: reporting
```
</File>


これにより、生成されたリレーションは `reporting` データベースに配置されるため、完全なリレーション名はデフォルトのターゲット データベースではなく `reporting.finance.sales_metrics` になります。

</TabItem>

<TabItem value="seeds" label="Seeds">

`dbt_project.yml` ファイルでデータベースを設定します。

例えば、シードをターゲットデータベースではなく `staging` というデータベースにロードするには、次のように設定します。

<File name='dbt_project.yml'>

```yml
seeds:
  your_project:
    product_categories:
      +database: staging
```

これにより、生成されたリレーションは `staging` データベースに配置されるため、完全なリレーション名は `staging.finance.product_categories` になります。

</File>
</TabItem>

<TabItem value="snapshots" label="Snapshots">

<VersionBlock lastVersion="1.8">

Available for dbt Cloud release tracks or dbt Core v1.9+. Select v1.9 or newer from the version dropdown to view the configs.

</VersionBlock>

<VersionBlock firstVersion="1.9">

`dbt_project.yml`、snapshot.yml ファイル、または設定ファイルで、スナップショットの保存先となるカスタムデータベースを指定します。

例えば、ターゲットデータベース以外のデータベースにスナップショットをロードしたい場合は、次のように設定します。

<File name='dbt_project.yml'>

```yml
snapshots:
  your_project:
    your_snapshot:
      +database: snapshots
```
</File>

または `snapshot_name.yml` ファイルで:

<File name='snapshots/snapshot_name.yml'>

```yaml
version: 2

snapshots:
  - name: snapshot_name
    [config](/reference/resource-properties/config):
      database: snapshots
```
</File>

この結果、生成されたリレーションは `snapshots` データベースに配置されるため、完全なリレーション名はデフォルトのターゲット データベースではなく `snapshots.finance.your_snapshot` になります。

</VersionBlock>

</TabItem>



<TabItem value="test" label="Tests">

`dbt_project.yml` ファイルで、テスト結果を保存するデータベースをカスタマイズします。

例えば、テスト結果を特定のデータベースに保存するには、次のように設定します:

<File name='dbt_project.yml'>

```yml
tests:
  +store_failures: true
  +database: test_results
```

これにより、テスト結果が `test_results` データベースに保存されます。

</File>
</TabItem>
</Tabs>


## 定義

オプションで、[モデル](/docs/build/sql-models)、[シード](/docs/build/seeds)、[スナップショット](/docs/build/snapshots)、または[データテスト](/docs/build/data-tests)のカスタムデータベースを指定します。

dbt がデータベースにリレーション (<Term id="table" />/<Term id="view" />) を作成する場合、`{{ database }}.{{ schema }}.{{ identifier }}` という形式で作成します (例: `analytics.finance.payments`)。

dbt の標準的な動作は次のとおりです。
* カスタムデータベースが指定されていない場合、リレーションのデータベースはターゲットデータベース (`{{ target.database }}`) になります。
* カスタムデータベースが指定されている場合、リレーションのデータベースは `{{ database }}` の値になります。

dbt がリレーションの `database` を生成する方法を変更する方法の詳細については、[カスタム データベースの使用](/docs/build/custom-databases) を参照してください。

## ウェアハウス固有の情報
* BigQuery: 「プロジェクト」と「データベース」は互換性があります

