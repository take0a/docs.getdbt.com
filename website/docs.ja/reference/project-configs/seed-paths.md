---
datatype: [directorypath]
default_value: [data]
---

<File name='dbt_project.yml'>

```yml
seed-paths: [directorypath]
```

</File>

## 定義
オプションで、[seed](/docs/build/seeds)ファイルが配置されているディレクトリのカスタムリストを指定します。

## デフォルト

デフォルトでは、dbt はシードが `seeds` ディレクトリにあるものと想定します。例: `seed-paths: ["seeds"]`。

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="seed-paths"
absolute="/Users/username/project/seed"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    seed-paths: ["seed"]
    ```

- ❌ **Don't:**
  - 絶対パスは避けてください:
    ```yml
    seed-paths: ["/Users/username/project/seed"]
    ```

## 例
### `seeds` の代わりに `custom_seeds` という名前のディレクトリを使用します

<File name='dbt_project.yml'>

```yml
seed-paths: ["custom_seeds"]
```

</File>

### モデルとシードを `models` ディレクトリに同じ場所に配置します。
注: dbt はシード（`.csv` ファイル）とモデル（`.sql` ファイル）に対して異なるファイルタイプを検索するため、この方法が機能します。

<File name='dbt_project.yml'>

```yml
seed-paths: ["models"]
model-paths: ["models"]
```

</File>

### シードを2つのディレクトリに分割します
注: 同様の効果を得るには、代わりに `seeds/` ディレクトリ内に2つのサブディレクトリを使用することをお勧めします。

<File name='dbt_project.yml'>

```yml
seed-paths: ["seeds", "custom_seeds"]
```

</File>
