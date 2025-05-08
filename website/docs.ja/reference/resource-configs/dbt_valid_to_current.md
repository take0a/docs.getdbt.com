---
resource_types: [snapshots]
description: "`dbt_valid_to_current` 設定を使用して、現在の snapshot  レコードの `dbt_valid_to` の値のカスタム インジケーターを設定します。"
datatype: "{<dictionary>}"
default_value: {NULL}
id: "dbt_valid_to_current"
---

<VersionCallout version="1.9" />

<File name='snapshots/schema.yml'>

```yaml
snapshots:
  - name: my_snapshot
    config:
      dbt_valid_to_current: "string"

```

</File>

<File name='snapshots/<filename>.sql'>

```sql
{{
    config(
        unique_key='id',
        strategy='timestamp',
        updated_at='updated_at',
        dbt_valid_to_current='string'
    )
}}
```

</File>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +dbt_valid_to_current: "string"
```

</File>

## 説明

`dbt_valid_to_current` 設定を使用して、現在の snapshot レコードの `dbt_valid_to` の値（将来の日付など）にカスタムインジケーターを設定します。デフォルトでは、この値は `NULL` です。設定すると、dbt は snapshot テーブル内の現在のレコードの `dbt_valid_to` に `NULL` ではなく、この指定された値を使用します。

この方法により、カスタム日付の割り当て、結合の操作、終了日を必要とする範囲ベースのフィルタリングの実行が容易になります。

:::warning

意図しないデータ変更を防ぐため、dbt は既存の `dbt_valid_to` 列の現在の値を自動的に調整しません。既存の現在のレコードの `dbt_valid_to` は引き続き `NULL` に設定されます。

`dbt_valid_to_current` 設定を適用した後に挿入された新しいレコードの `dbt_valid_to` は、デフォルトの `NULL` 値ではなく、指定された値（「9999-12-31」など）に設定されます。

:::

### 考慮事項

- **日付式** - データプラットフォームと互換性のあるハードコードされた日付式（例：`to_date('9999-12-31')`）を使用してください。構文はウェアハウスによって異なる場合がありますのでご注意ください（例：`to_date('YYYY-MM-DD'`）または`date(YYYY, MM, DD)`）。

- **Jinja の制限事項** - `dbt_valid_to_current` は静的 SQL 式のみを受け入れます。Jinja 式（例：`{{ var('my_future_date') }}`）はサポートされていません。

- **遅延と `state:modified`** - `dbt_valid_to_current` への変更は、遅延および `--select state:modified` と互換性があります。この構成が変更されると、`state:modified` 選択に表示され、必要な snapshot の更新を手動で行うように警告が表示されます。

## デフォルト

デフォルトでは、 snapshot テーブル内の現在の（最新の）レコードの `dbt_valid_to` は `NULL` に設定されています。つまり、これらのレコードは引き続き有効であり、終了日は定義されていません。

現在および将来のレコードの `dbt_valid_to` に `NULL` ではなく特定の値を使用したい場合は、`dbt_valid_to_current` 設定オプションを使用できます。例えば、遠い将来の日付（`9999-12-31`）を設定できます。

`dbt_valid_to_current` に割り当てる値は、データベースの要件に応じて、有効な日付またはタイムスタンプを表す文字列である必要があります。データプラットフォーム内で機能する式を使用してください。


##  snapshot レコードへの影響

`dbt_valid_to_current` を設定すると、dbt が snapshot テーブル内の `dbt_valid_to` 列を管理する方法に影響します。

- **既存レコードの場合** - 意図しないデータ変更を防ぐため、dbt は既存の `dbt_valid_to` 列の現在の値を自動的に調整しません。既存の現在のレコードの `dbt_valid_to` は引き続き `NULL` に設定されます。

- **新規レコードの場合** - `dbt_valid_to_current` 設定を適用した後に挿入された新しいレコードでは、`dbt_valid_to` は `NULL` ではなく、指定された値（例: '9999-12-31'）に設定されます。

つまり、 snapshot テーブルには、`dbt_valid_to` 値が `NULL`（既存データ）と新しく指定された値（新規データ）の両方である現在のレコードが含まれることになります。現在のレコードに対して一貫した `dbt_valid_to` 値を保持したい場合は、 snapshot  テーブル内の既存のレコード (`dbt_valid_to` が `NULL` の場合) を手動で更新して、`dbt_valid_to_current` 値と一致させることができます。

## 例

<File name='snapshots/schema.yml'>

```yaml
snapshots:
  - name: my_snapshot
    config:
      strategy: timestamp
      updated_at: updated_at
      dbt_valid_to_current: "to_date('9999-12-31')"
    columns:
      - name: dbt_valid_from
        description: The timestamp when the record became valid.
      - name: dbt_valid_to
        description: >
          The timestamp when the record ceased to be valid. For current records,
          this is either `NULL` or the value specified in `dbt_valid_to_current`
          (like `'9999-12-31'`).
```

</File>

結果の snapshot テーブルには、構成された dbt_valid_to 列の値が含まれます:

| id | dbt_scd_id           |    dbt_updated_at    |       dbt_valid_from |     dbt_valid_to     |
| -- | -------------------- | -------------------- | -------------------- | -------------------- |
|  1 | 60a1f1dbdf899a4dd... | 2024-10-02 ...       | 2024-10-02 ...       | 9999-12-31 ...       |
|  2 | b1885d098f8bcff51... | 2024-10-02 ...       | 2024-10-02 ...       | 9999-12-31 ...       |
