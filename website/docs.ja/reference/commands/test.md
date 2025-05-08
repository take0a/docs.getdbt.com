---
title: "dbt test コマンドについて"
sidebar_label: "test"
id: "test"
---
<VersionBlock lastVersion="1.7">

`dbt test` は、モデル、ソース、スナップショット、シードに対して定義されたテストを実行します。これらのリソースは、適切なコマンドを使用して既に作成されていることを前提としています。

実行するテストは、[こちら](/reference/node-selection/syntax) で説明されている `--select` フラグを使用して選択できます。

```bash
# run tests for one_specific_model
dbt test --select "one_specific_model"

# run tests for all models in package
dbt test --select "some_package.*"

# run only tests defined singularly
dbt test --select "test_type:singular"

# run only tests defined generically
dbt test --select "test_type:generic"

# run singular tests limited to one_specific_model
dbt test --select "one_specific_model,test_type:singular"

# run generic tests limited to one_specific_model
dbt test --select "one_specific_model,test_type:generic"
```

テストの作成方法の詳細については、[テストのドキュメント](/docs/build/data-tests)を参照してください。

</VersionBlock>

<VersionBlock firstVersion="1.8">

`dbt test` は、モデル、ソース、スナップショット、シードに対して定義されたデータテストと、SQL モデルに対して定義された単体テストを実行します。これらのリソースは、適切なコマンドを使用して既に作成されていることを前提としています。

実行するテストは、[こちら](/reference/node-selection/syntax) で説明されている `--select` フラグを使用して選択できます。

```bash
# run data and unit tests
dbt test

# run only data tests
dbt test --select test_type:data

# run only unit tests
dbt test --select test_type:unit

# run tests for one_specific_model
dbt test --select "one_specific_model"

# run tests for all models in package
dbt test --select "some_package.*"

# run only data tests defined singularly
dbt test --select "test_type:singular"

# run only data tests defined generically
dbt test --select "test_type:generic"

# run data tests limited to one_specific_model
dbt test --select "one_specific_model,test_type:data"

# run unit tests limited to one_specific_model
dbt test --select "one_specific_model,test_type:unit"
```

テストの記述方法の詳細については、[データ テスト](/docs/build/data-tests) および [ユニット テスト](/docs/build/unit-tests) のドキュメントをお読みください。

</VersionBlock>



