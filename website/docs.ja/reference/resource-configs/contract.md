---
resource_types: [models]
description: "契約構成が強制されると、dbt は、モデルによって返されるデータセットが、name や data_type などの yaml で定義した属性や、データ プラットフォームでサポートされている追加の制約と完全に一致することを確認します。"
datatype: "{<dictionary>}"
default_value: {enforced: false}
id: "contract"
---

`contract` 構成が適用されると、dbt はモデルが返すデータセットが、yaml で定義した属性と完全に一致するようにします。
- すべての列の `name` と `data_type`
- このマテリアライゼーションとデータプラットフォームでサポートされている追加の [`constraints`](/reference/resource-properties/constraints)

これは、dbt の内外を問わず、下流のモデルに対してクエリを実行するユーザーが、分析に使用する列セットを予測可能かつ一貫性のあるものにするためです。`boolean` (`true`/`false`) から `integer` (`0`/`1`) へのデータ型のわずかな変更でさえ、予期せぬ形でクエリが失敗する可能性があります。

## サポート

現在、モデルコントラクトは以下のモデルでサポートされています。
- SQL モデル（Python はまだサポートされていません）
- `table`、`view`、`incremental` としてマテリアライズされたモデル（`on_schema_change: append_new_columns` または `on_schema_change: fail` を使用）
- 最も一般的なデータプラットフォーム（ただし、さまざまな [制約タイプ](/reference/resource-properties/constraints) のサポートと適用はプラットフォームによって異なります）

## データ型エイリアス

dbt は、YAML で定義された `data_type` に対して組み込みの型エイリアスを使用します。例えば、コントラクトで `string` を指定すると、Postgres/Redshift では dbt によって `text` に変換されます。`data_type` 名が既知のエイリアスに含まれていない場合、dbt はそれをそのまま渡します。これはデフォルトで有効になっていますが、`alias_types` を `false` に設定することで無効にできます。

無効化の例:

<File name='FOLDER_NAME/FILE_NAME.yml'>

```yml

models:
  - name: my_model
    config:
      contract:
        enforced: true
        alias_types: false  # true by default

```

</File>

## サイズ、精度、スケール

dbt はデータ型を比較す​​る際に、サイズ、精度、スケールといった詳細な比較は行いません。`varchar(256)` と `varchar(257)` の違いは、下流のクエリ実行者のエクスペリエンスにほとんど影響を与えないため、あまり気にする必要はないと考えています。[カスタムテストを作成または使用](/best-practices/writing-custom-generic-tests) することで、より正確なアサーションを実現できます。

varchar のサイズまたは数値スケールを指定する必要があります。指定しない場合、dbt はデフォルト値を使用します。たとえば、`numeric` 型のデフォルト値が精度 38、スケール 0 の場合、数値列には小数点以下の桁数が 0 となり（整数のみが格納されます）、コントラクトの適用に失敗する可能性があります。この暗黙的な強制を回避するには、`numeric(38, 6)` のように、`data_type` をゼロ以外のスケールで指定します。dbt Core 1.7 以降では、数値データ型を指定するときに精度とスケールを指定しないと警告が表示されます。

### 例

<File name='models/dim_customers.yml'>

```yml
models:
  - name: dim_customers
    config:
      materialized: table
      contract:
        enforced: true
    columns:
      - name: customer_id
        data_type: int
        constraints:
          - type: not_null
      - name: customer_name
        data_type: string
      - name: non_integer
        data_type: numeric(38,3)
```

</File>

モデルが次のように定義されているとします:

<File name='models/dim_customers.sql'>

```sql
select
  'abc123' as customer_id,
  'My Best Customer' as customer_name
```

</File>

モデルを `dbt run` すると、dbt がそれをデータベース内のテーブルとして実現する前に、次のエラーが表示されます:

```txt
20:53:45  Compilation Error in model dim_customers (models/dim_customers.sql)
20:53:45    This model has an enforced contract that failed.
20:53:45    Please ensure the name, data_type, and number of columns in your contract match the columns in your model's definition.
20:53:45
20:53:45    | column_name | definition_type | contract_type | mismatch_reason    |
20:53:45    | ----------- | --------------- | ------------- | ------------------ |
20:53:45    | customer_id | TEXT            | INT           | data type mismatch |
20:53:45
20:53:45
20:53:45    > in macro assert_columns_equivalent (macros/materializations/models/table/columns_spec_ddl.sql)
```


### 増分モデルと `on_schema_change`

増分モデルでも [`on_schema_change`](/docs/build/incremental-models#what-if-the-columns-of-my-incremental-model-change) の設定が必要なのはなぜですか？また、`append_new_columns` または `fail` が必要なのはなぜですか？

想像してみてください。
- SQL と YAML の両方の仕様に新しい列を追加します。
- `on_schema_change` を設定しないか、`on_schema_change: 'ignore'` を設定します。
- dbt は実際には新しい列を既存のテーブルに追加しません。それでも upsert/merge は成功します。これは、既存の「宛先」列のみに基づいて upsert/merge を実行するためです（これは長年確立された動作です）。
- 結果として、yaml で定義されたコントラクトとデータベース内の実際のテーブルとの間に差分が生じます。つまり、コントラクトが正しくないということです！

なぜ `sync_all_columns` ではなく `append_new_columns` (または `fail`) を使用するのでしょうか？既存の列を削除すると、コントラクトモデルにとって互換性のない変更となるためです。 `sync_all_columns` は `append_new_columns` と同様に機能しますが、削除された列も削除します。これは、バージョンをアップグレードしない限り、縮小モデルでは実行されないはずです。

## 関連ドキュメント
- [モデル契約とは](/docs/collaborate/govern/model-contracts)
- [`columns` の定義](/reference/resource-properties/columns)
- [`constraints` の定義](/reference/resource-properties/constraints)
