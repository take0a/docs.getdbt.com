---
resource_types: sources
datatype: string
---

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <source_name>
    overrides: <package name>

    database: ...
    schema: ...
```

</File>

## 定義

インクルードされたパッケージで定義されたソースをオーバーライドします。オーバーライド元のソースで定義されたプロパティは、オーバーライド先のソースの基本プロパティの上に適用されます。

以下のソースプロパティをオーバーライドできます:
 - [description](/reference/resource-properties/description)
 - [meta](/reference/resource-configs/meta)
 - [database](/reference/resource-properties/database)
 - [schema](/reference/resource-properties/schema)
 - [loader](/reference/resource-properties/loader)
 - [quoting](/reference/resource-properties/quoting)
 - [freshness](/reference/resource-properties/freshness)
 - [loaded_at_field](/reference/resource-properties/freshness#loaded_at_field)
 - [tags](/reference/resource-configs/tags)

## 例

### パッケージで定義されたソースのデータベース名とスキーマ名を指定します

この例は、[Fivetran GitHub ソースパッケージ](https://github.com/fivetran/dbt_github_source/blob/830ba43ac2948e4853a3c167ab7ee88b8b425fa0/models/src_github.yml#L3-L29) に基づいています。
ここでは、`github_source` パッケージを含む親 dbt プロジェクトでデータベースとスキーマがオーバーライドされています。

<File name='models/src_github.yml'>

```yml
version: 2

sources:
  - name: github
    overrides: github_source

    database: RAW
    schema: github_data

```

</File>

### パッケージ内のソーステーブルに対して、独自のソース鮮度を設定します。

ソースレベルと <Term id="table" /> レベルの両方で設定をオーバーライドできます。

<File name='models/src_github.yml'>

```yml
version: 2

sources:
  - name: github
    overrides: github_source

    freshness:
      warn_after:
        count: 1
        period: day
      error_after:
        count: 2
        period: day

    tables:
      - name: issue_assignee
        freshness:
          warn_after:
            count: 2
            period: day
          error_after:
            count: 4
            period: day

```

</File>
