---
resource_types: [models]
datatype: model_name
required: yes
---

<File name='models/<schema>.yml'>

```yml
version: 2

models:
  - name: model_name
```

</File>

## 定義

プロパティを宣言するモデルの名前。モデルの_ファイル名_と一致する必要があります（大文字と小文字の区別を含む）。大文字と小文字が一致しないと、dbt が設定を正しく適用できず、dbt Explorer のメタデータに影響する可能性があります。

## デフォルト

これは**必須プロパティ**であり、デフォルトは存在しません。
