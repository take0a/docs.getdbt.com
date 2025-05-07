---
datatype: [directorypath]
default_value: [test]
---

<File name='dbt_project.yml'>

```yml
test-paths: [directorypath]
```

</File>

## 定義

オプションで、[特異テスト](/docs/build/data-tests#singular-data-tests)と[カスタム汎用テスト](/docs/build/data-tests#generic-data-tests)が配置されているディレクトリのカスタムリストを指定します。


## デフォルト
この設定を指定しない場合、dbt は `tests` ディレクトリ（つまり `test-paths: ["tests"]`）内のテストを検索します。具体的には、以下の内容を含む `.sql` ファイルを検索します。
- `tests/generic` サブディレクトリ内の汎用テスト定義
- 個別テスト（その他のファイル）

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="test-paths"
absolute="/Users/username/project/test"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    test-paths: ["test"]
    ```

- ❌ **Don't:**
  - 絶対パスは避けてください:
    ```yml
    test-paths: ["/Users/username/project/test"]
    ```

## 例
### データテストには `tests` ではなく `custom_tests` というサブディレクトリを使用します

<File name='dbt_project.yml'>

```yml
test-paths: ["custom_tests"]
```

</File>
