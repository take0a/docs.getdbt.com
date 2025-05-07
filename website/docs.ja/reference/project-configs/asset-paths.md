---
datatype: [directorypath]
description: "dbt の asset-paths 構成を理解するには、このガイドをお読みください。"
default_value: []
---

<File name='dbt_project.yml'>

```yml
asset-paths: [directorypath]
```

</File>

## 定義
`docs generate` コマンドの一部として、`target` ディレクトリにコピーするディレクトリのカスタムリストをオプションで指定します。これは、リポジトリ内の画像をプロジェクトドキュメントにレンダリングする場合に便利です。


## デフォルト

デフォルトでは、dbt はドキュメント生成時に追加のファイルをコピーしません。例えば、`asset-paths: []` などです。

import RelativePath from '/snippets.ja/_relative-path.md';

<RelativePath 
path="asset-paths"
absolute="/Users/username/project/assets"
/>

- ✅ **Do**
  - 相対パスを使用:
    ```yml
    asset-paths: ["assets"]
    ```

- ❌ **Don't**
  - 絶対パスは避けてください:
    ```yml
    asset-paths: ["/Users/username/project/assets"]
    ```

## 例
### `docs generate` の一部として `assets` サブディレクトリ内のファイルをコンパイルします。

<File name='dbt_project.yml'>

```yml
asset-paths: ["assets"]
```

</File>

このディレクトリに含まれるファイルはすべて、`dbt docs generate` によって `target/` ディレクトリにコピーされ、プロジェクトドキュメント内で画像としてアクセスできるようになります。

説明に画像を含める方法の詳細については、[こちら](/reference/resource-properties/description/#include-an-image-from-your-repo-in-your-descriptions) をご覧ください。
