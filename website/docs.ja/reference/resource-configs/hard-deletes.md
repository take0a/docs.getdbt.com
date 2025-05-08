---
title: hard_deletes
resource_types: [snapshots]
description: "`hard_deletes` 設定を使用して、 snapshot  テーブルで削除された行を追跡する方法を制御します。"
datatype: "boolean"
default_value: {ignore}
id: "hard-deletes"
sidebar_label: "hard_deletes"
---

<VersionCallout version="1.9" />

<File name='snapshots/schema.yml'>

```yaml
snapshots:
  - name: <snapshot_name>
    config:
      hard_deletes: 'ignore' | 'invalidate' | 'new_record'
```
</File>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +hard_deletes: "ignore" | "invalidate" | "new_record"
```

</File>

<File name='snapshots/<filename>.sql'>

```sql
{{
    config(
        unique_key='id',
        strategy='timestamp',
        updated_at='updated_at',
        hard_deletes='ignore' | 'invalidate' | 'new_record'
    )
}}
```

</File>


## 説明

`hard_deletes` 設定を使用すると、ソースから削除された行の処理方法をより詳細に制御できます。サポートされているオプションは、`ignore`（デフォルト）、`invalidate`（従来の `invalidate_hard_deletes=true` の置き換え）、`new_record` です。`new_record` は、 snapshot テーブルに新しいメタデータ列を作成することに注意してください。

`hard_deletes` は、dbt-postgres、dbt-bigquery、dbt-snowflake、および dbt-redshift アダプタで使用できます。

import HardDeletes from '/snippets.ja/_hard-deletes.md';

<HardDeletes />

:::warning

既存の snapshot を `hard_deletes` 設定を使用して更新する場合、dbt は移行を自動的に処理しません。これらの設定は新規の snapshot にのみ使用するか、この設定を有効にする前に既存のテーブルを[更新](/reference/snapshot-configs#snapshot-configuration-migration)することをお勧めします。
:::

## デフォルト

デフォルトでは、`hard_deletes` を指定しない場合、自動的に `ignore` が使用されます。削除された行は追跡されず、`dbt_valid_to` 列は `NULL` のままになります。

`hard_deletes` 設定には 3 つのメソッドがあります。

| Methods | Description |
| --------- | ----------- |
| `ignore` (default) | 削除されたレコードに対してはアクションはありません。 |
| `invalidate` | 既存の `invalidate_hard_deletes=true` と同じように動作します。`dbt_valid_to` を現在の時刻に設定することで、削除されたレコードが無効化されます。このメソッドは `invalidate_hard_deletes` 設定に代わるもので、ソースから削除された行の処理方法をより細かく制御できます。 |
| `new_record` | レコードが削除されたときに、`dbt_is_deleted` メタ フィールドを使用して、削除されたレコードを新しい行として追跡します。 |

## 考慮事項

- **後方互換性**: `invalidate_hard_deletes` 設定は既存の snapshot では引き続きサポートされますが、`hard_deletes` と併用することはできません。
- **新しい snapshot **: 新しい snapshot では、`invalidate_hard_deletes` ではなく `hard_deletes` を使用することをお勧めします。
- **移行**: 既存の snapshot を、データを移行せずに `hard_deletes` を使用するように切り替えた場合、古いデータ形式と新しいデータ形式が混在するなど、一貫性のない結果や誤った結果が発生する可能性があります。

## 例

<File name='snapshots/schema.yml'>

```yaml
snapshots:
  - name: my_snapshot
    config:
      hard_deletes: new_record  # options are: 'ignore', 'invalidate', or 'new_record'
      strategy: timestamp
      updated_at: updated_at
    columns:
      - name: dbt_valid_from
        description: Timestamp when the record became valid.
      - name: dbt_valid_to
        description: Timestamp when the record stopped being valid.
      - name: dbt_is_deleted
        description: Indicates whether the record was deleted.
```

</File>

結果の snapshot テーブルには、`hard_deletes: new_record` 設定が含まれます。レコードが削除され、後で復元された場合、結果の snapshot テーブルは次のようになります:

| id | dbt_scd_id           |   Status | dbt_updated_at       |   dbt_valid_from    |     dbt_valid_to     | dbt_is_deleted | 
| -- | -------------------- | -----    | -------------------- | --------------------| -------------------- | ----------- |
|  1 | 60a1f1dbdf899a4dd... | pending  | 2024-10-02 ...       | 2024-05-19...       | 2024-05-20 ...       | False       | 
|  1 | b1885d098f8bcff51... | pending  | 2024-10-02 ...       | 2024-05-20 ...      | 2024-06-03 ...       | True        | 
|  1 | b1885d098f8bcff53... | shipped  | 2024-10-02 ...       | 2024-06-03 ...      |                      | False       | 
|  2 | b1885d098f8bcff55... | active   | 2024-10-02 ...       | 2024-05-19 ...      |                      | False       | 
 
この例では、レコードが削除されると `dbt_is_deleted` 列が `True` に設定されます。レコードが復元されると、 `dbt_is_deleted` 列は `False` に設定されます。
