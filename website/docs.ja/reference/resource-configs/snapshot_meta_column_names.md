---
resource_types: [snapshots]
description: "Snapshot meta column names"
datatype: "{<dictionary>}"
default_value: {"dbt_valid_from": "dbt_valid_from", "dbt_valid_to": "dbt_valid_to", "dbt_scd_id": "dbt_scd_id", "dbt_updated_at": "dbt_updated_at"}
id: "snapshot_meta_column_names"
---

<VersionCallout version="1.9" />

<File name='snapshots/schema.yml'>

```yaml
snapshots:
  - name: <snapshot_name>
    config:
      snapshot_meta_column_names:
        dbt_valid_from: <string>
        dbt_valid_to: <string>
        dbt_scd_id: <string>
        dbt_updated_at: <string>
        dbt_is_deleted: <string>

```

</File>

<File name='snapshots/<filename>.sql'>

```jinja2
{{
    config(
      snapshot_meta_column_names={
        "dbt_valid_from": "<string>",
        "dbt_valid_to": "<string>",
        "dbt_scd_id": "<string>",
        "dbt_updated_at": "<string>",
        "dbt_is_deleted": "<string>",
      }
    )
}}

```

</File>

<File name='dbt_project.yml'>

```yml
snapshots:
  [<resource-path>](/reference/resource-configs/resource-path):
    +snapshot_meta_column_names:
      dbt_valid_from: <string>
      dbt_valid_to: <string>
      dbt_scd_id: <string>
      dbt_updated_at: <string>
      dbt_is_deleted: <string>
```

</File>

## 説明

組織の命名規則に合わせるために、`snapshot_meta_column_names` 設定を使用して、各スナップショット内の [メタデータ列](/docs/build/snapshots#snapshot-meta-fields) の名前をカスタマイズできます。

## デフォルト

デフォルトでは、dbtスナップショットは[タイプ2 緩やかに変化するディメンション](https://en.wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row)レコードを使用して変更履歴を追跡するために、以下の列名を使用します。

| Field          | <div style={{width:'250px'}}>Meaning</div> | Notes | Example |
| -------------- | ------- | ----- | ------- |
| `dbt_valid_from` | このスナップショット行が最初に挿入され、有効になったときのタイムスタンプ。| 値は [`strategy`](/reference/resource-configs/strategy) の影響を受けます。 | `snapshot_meta_column_names: {dbt_valid_from: start_date}` |
| `dbt_valid_to`   | この行が有効でなくなったときのタイムスタンプ。 |  | `snapshot_meta_column_names: {dbt_valid_to: end_date}` |
| `dbt_scd_id`     | 各スナップショット行に対して生成される一意のキー。 | これは dbt によって内部的に使用されます。 | `snapshot_meta_column_names: {dbt_scd_id: scd_id}` |
| `dbt_updated_at` | このスナップショット行が挿入されたときのソース レコードの `updated_at` タイムスタンプ。 | これは dbt によって内部的に使用されます。 | `snapshot_meta_column_names: {dbt_updated_at: modified_date}` |
| `dbt_is_deleted` | レコードが削除されたかどうかを示す文字列値。(削除された場合は `True`、削除されていない場合は `False`)。|`hard_deletes='new_record'` が設定されている場合に追加されます。  | `snapshot_meta_column_names: {dbt_is_deleted: is_deleted}` |

これらの列名はすべて、`snapshot_meta_column_names` 設定を使用してカスタマイズできます。詳細については、[例](#example) を参照してください。

:::warning  

意図しないデータ変更を防ぐため、dbt は列名の変更を自動的には適用しません。そのため、ユーザーが既存のテーブルを更新せずにスナップショットに `snapshot_meta_column_names` 設定を適用すると、エラーが発生します。これらの設定は新規のスナップショットにのみ使用するか、列名の変更をコミットする前に既存のテーブルを更新することをお勧めします。

:::

## Example

<File name='snapshots/schema.yml'>

```yaml
snapshots:
  - name: orders_snapshot
    relation: ref("orders")
    config:
      unique_key: id
      strategy: check
      check_cols: all
      hard_deletes: new_record
      snapshot_meta_column_names:
        dbt_valid_from: start_date
        dbt_valid_to: end_date
        dbt_scd_id: scd_id
        dbt_updated_at: modified_date
        dbt_is_deleted: is_deleted
```

</File>

結果のスナップショット テーブルには、構成されたメタ列名が含まれます。

| id | scd_id               |        modified_date |           start_date |             end_date | is_deleted |
| -- | -------------------- | -------------------- | -------------------- | -------------------- | ---------- |
|  1 | 60a1f1dbdf899a4dd... | 2024-10-02 ...       | 2024-10-02 ...       | 2024-10-03 ...       | False      |
|  1 | 60a1f1dbdf899a4dd... | 2024-10-03 ...       | 2024-10-03 ...       |                      | True      |
|  2 | b1885d098f8bcff51... | 2024-10-02 ...       | 2024-10-02 ...       |                      | False     |
