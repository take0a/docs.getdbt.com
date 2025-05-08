---
resource_types: sources
datatype: {dictionary}
---

<File name='models/<filename>.yml'>

```yml
version: 2

sources:
  - name: <source_name>
    tables:
      - name: <table_name>
        external:
          location: <string>
          file_format: <string>
          row_format: <string>
          tbl_properties: <string>      
          partitions:
            - name: <column_name>
              data_type: <string>
              description: <string>
              meta: {dictionary}
            - ...
          <additional_property>: <additional_value>
```

</File>

## 定義

外部テーブルを参照するソースに固有のメタデータプロパティの拡張可能なディクショナリです。
Hive の外部 <Term id="table" /> 仕様にほぼ対応する、シンプルな型検証を備えたオプションの組み込みプロパティがあります。必要な数だけ追加プロパティを定義して使用できます。

`external` プロパティを定義すると、次のようなことが可能になります。
- [`graph.sources`](/reference/dbt-jinja-functions/graph) をイントロスペクトするマクロを強化する
- 後で [manifest](/reference/artifacts/manifest-json) から抽出できるメタデータを定義する

このプロパティを使用してカスタムワークフローを強化する方法の例については、[`dbt-external-tables`](https://github.com/dbt-labs/dbt-external-tables) パッケージを参照してください。
