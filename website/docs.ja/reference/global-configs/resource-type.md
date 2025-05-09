---
title: "Resource type"
id: "resource-type"
sidebar: "resource type"
---

<VersionBlock lastVersion="1.8">

The `--resource-type` and `--exclude-resource-type` flags include or exclude resource types from the `dbt build`, `dbt clone`, and `dbt list` commands. In dbt v1.9 onwards, these flags are also supported in the `dbt test` command.

</VersionBlock>

<VersionBlock firstVersion="1.9">

`--resource-type` フラグと `--exclude-resource-type` フラグは、`dbt build`、`dbt test`、`dbt clone`、および `dbt list` コマンドにリソース タイプを含めるか除外します。

</VersionBlock>

つまり、フラグを使用すると、特定のリソースをターゲットにするのではなく、コマンドを実行するときに含めるまたは除外するリソースの種類を指定できます。

:::tip Note
`--exclude-resource-type` フラグは、dbt バージョン 1.8 以降でのみ使用できます。それより古いバージョンをご利用の場合、このフラグは使用できません。
:::

利用可能なリソースの種類は次のとおりです:

<VersionBlock lastVersion="1.7">

- [`analysis`](/docs/build/analyses)
- [`exposure`](/docs/build/exposures)
- [`metric`](/docs/build/build-metrics-intro)
- [`model`](/docs/build/models)
- [`saved_query`](/docs/build/saved-queries)
- [`seed`](/docs/build/seeds)
- [`semantic_model`](/docs/build/semantic-models)
- [`snapshot`](/docs/build/snapshots)
- [`source`](/docs/build/sources)
- [`test`](/docs/build/data-tests)

</VersionBlock>

<VersionBlock firstVersion="1.8">

- [`analysis`](/docs/build/analyses)
- [`exposure`](/docs/build/exposures)
- [`metric`](/docs/build/build-metrics-intro)
- [`model`](/docs/build/models)
- [`saved_query`](/docs/build/saved-queries)
- [`seed`](/docs/build/seeds)
- [`semantic_model`](/docs/build/semantic-models)
- [`snapshot`](/docs/build/snapshots)
- [`source`](/docs/build/sources)
- [`test`](/docs/build/data-tests)
- [`unit_test`](/docs/build/unit-tests)

</VersionBlock>

## 例

特定のリソースをターゲットにする代わりに、`--resource-flag` または `--exclude-resource-type` フラグを使用して、特定のタイプのすべてのリソースをターゲットにします。`dbt build --resource-type RESOURCE_TYPE` で、`RESOURCE_TYPE` は含めたいリソースタイプに置き換えてください。

- たとえば、dbt ビルドプロセスからすべてのスナップショットを含めるには、次のコマンドを使用します:

    <File name='Usage'>

    ```text
    dbt build --resource-type snapshot
    ```

    </File>


- この例では、次のコマンドを実行して、`--resource-type` フラグを使用して保存されたすべてのクエリを含めます:

    <File name='Usage'>

    ```text
    dbt build --resource-type saved_query
    ```

    </File>

<VersionBlock firstVersion="1.8">

- この例では、以下のコマンドを使用して、dbt ビルドプロセスからすべてのユニットテストを除外します。`--exclude-resource-type` フラグは dbt バージョン 1.8 以降でのみ使用可能です:

    <File name='Usage'>

    ```text
    dbt build --exclude-resource-type unit_test
    ```

    </File>

- この例では、次のコマンドを使用して、すべてのデータ テストをビルド プロセスに含めます:

    <File name='Usage'>

    ```text
    dbt build --resource-type test
    ```

    </File>

</VersionBlock>

<VersionBlock firstVersion="1.9">

- この例では、テストの実行時にすべての単体テストを除外するには、次のコマンドを使用します:

    <File name='Usage'>

    ```text
    dbt test --exclude-resource-type unit_test
    ```

    </File>

- この例では、テストを実行するときにすべてのデータ テストを含めるために次のコマンドを使用します:

    <File name='Usage'>

    ```text
    dbt test --resource-type test
    ```

    </File>

</VersionBlock>
