---
datatype: [directorypath]
default_value: [models]
---

<File name='dbt_project.yml'>

```yml
model-paths: [directorypath]
```

</File>

## 定義
オプションで、[モデル](/docs/build/models)、[ソース](/docs/build/sources)、[ユニットテスト](/docs/build/unit-tests)が配置されているディレクトリのカスタムリストを指定します。

## デフォルト
デフォルトでは、dbt はモデルとソースを `models` ディレクトリ内で検索します。たとえば、`model-paths: ["models"]` のようになります。

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="model-paths"
absolute="/Users/username/project/models"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    model-paths: ["models"]
    ```

- ❌ **Don't:**
  - 絶対パスは避けてください:
    ```yml
    model-paths: ["/Users/username/project/models"]
    ```

## 例
### `models` の代わりに `transformations` という名前のサブディレクトリを使用します

<File name='dbt_project.yml'>

```yml
model-paths: ["transformations"]
```

</File>
