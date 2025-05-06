---
title: 説明文に長い説明を書くにはどうすればいいですか?
description: "ドキュメントに長い説明を書く"
sidebar_label: '長い説明を書く'
id: long-descriptions
---

モデルの説明に1文では足りない場合は、以下の方法があります。

1. `>` を使って説明を複数行に分割します。行間の改行は削除され、Markdown が使用可能になります。この方法は、1段落だけのシンプルな説明に推奨されます。

```yml
  version: 2

  models:
  - name: customers
    description: >
      Lorem ipsum **dolor** sit amet, consectetur adipisicing elit, sed do eiusmod
      tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
      quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
      consequat.
```

2. `|` を使って説明を複数行に分割します。行内の改行は維持され、Markdown も使用できます。この方法は、より複雑な説明に推奨されます。

```yml
  version: 2

  models:
  - name: customers
    description: |
      ### Lorem ipsum

      * dolor sit amet, consectetur adipisicing elit, sed do eiusmod
      * tempor incididunt ut labore et dolore magna aliqua.
```

3. [docs ブロック](/docs/build/documentation#using-docs-blocks)を使用して、説明を別の Markdown ファイルに記述します。