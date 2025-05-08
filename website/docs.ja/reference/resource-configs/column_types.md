---
resource_types: [seeds]
datatype: {column_name: datatype}
---

## 説明

[seed](/docs/build/seeds) 内の列のデータベースタイプをオプションで指定します。キーが列名、値が有効なデータ型（データベースによって異なります）である辞書を指定します。

これを指定しない場合、dbt は seed ファイル内の列値に基づいてデータ型を推測します。

## 使用方法

`dbt_project.yml` ファイルで列タイプを指定します。

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    country_codes:
      +column_types:
        country_code: varchar(2)
        country_name: varchar(32)
```

</File>



Or:

<File name='seeds/properties.yml'>

```yml
version: 2

seeds:
  - name: country_codes
    config:
      column_types:
        country_code: varchar(2)
        country_name: varchar(32)
```

</File>

以前に `dbt seed` を実行したことがある場合は、変更を有効にするために `dbt seed --full-refresh` を実行する必要があります。

`column_types` を設定する際は、シードの完全なディレクトリパスを使用する必要があります。例えば、`seeds/marketing/utm_mappings.csv` にあるシードファイルの場合は、次のように設定する必要があります。

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    marketing:
      utm_mappings:
        +column_types:
          ...

```

</File>

## 例

### 郵便番号の先頭のゼロを保持するには、varchar 列型を使用します。

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop: # you must include the project name
    warehouse_locations:
      +column_types:
        zipcode: varchar(5)
```

</File>

## 推奨事項

この設定は、型推論が期待どおりに動作しないなど、必要な場合にのみ使用してください。それ以外の場合は、この設定を省略できます。

## トラブルシューティング

注: `column_types` 設定は、引用符の設定に関わらず、大文字と小文字が区別されます。シードで列を `Country_Name` として指定する場合は、`country_name` ではなく `Country_Name` として参照する必要があります。
