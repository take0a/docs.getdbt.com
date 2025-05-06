---
title: テストと説明を含む `.yml` ファイルの名前は `schema.yml` にする必要がありますか?
description: "テストと説明ファイルの命名"
sidebar_label: 'テストと説明ファイルの命名方法'
id: schema-yml-name

---
いいえ！このファイルの名前は自由に付けることができます（`whatever_you_want.yml` など）。ただし、以下の条件を満たしている必要があります。
* ファイルは `models/` ディレクトリ内にあります¹。
* ファイルの拡張子は `.yml` です。

詳しくは [ドキュメント](/reference/configs-and-properties) をご覧ください。

¹シード、スナップショット、またはマクロのプロパティを宣言する場合は、このファイルをそれぞれ `seeds/`、`snapshots/`、`macros/` などの関連ディレクトリに配置することもできます。
