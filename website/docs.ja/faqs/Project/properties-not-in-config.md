---
title: 構成ブロックにテストと説明を追加できますか?
description: "構成ブロックで定義されるプロパティの種類"
sidebar_label: '構成ブロックで定義されるプロパティの種類'
id: properties-not-in-config

---

dbt は、`config()` ブロックと `dbt_project.yml` に加えて、`.yml` ファイルでもノード設定を定義できます。しかし、その逆は必ずしも真ではありません。`.yml` ファイルには、そのファイルでしか定義できない項目もあります。

一部のプロパティは特別なものです。理由は以下のとおりです。
- 固有の Jinja レンダリング コンテキストを持つ
- 新しいプロジェクト リソースを作成する
- 階層的な構成としては意味をなさない
- まだ構成として再定義されていない古いプロパティである

これらのプロパティは次のとおりです。
- [`description`](/reference/resource-properties/description)
- [`tests`](/reference/resource-properties/data-tests)
- [`docs`](/reference/resource-configs/docs)
- `columns`
- [`quote`](/reference/resource-properties/columns#quote)
- [`source` プロパティ](/reference/source-properties) (例: `loaded_at_field`、`freshness`)
- [`exposure` プロパティ](/reference/exposure-properties) (例: `type`、 `maturity`)
- [`macro` プロパティ](/reference/macro-properties) (例: `arguments`)
