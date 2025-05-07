---
title: プロパティを定義する
sidebar_label: プロパティを定義する
intro_text: "properties.yml ファイルでリソースのプロパティを定義する方法を学びます"
description: "properties.yml ファイルでリソースのプロパティを定義する方法を学びます"
pagination_previous: "reference/define-configs"
---

dbt では、`properties.yml` ファイルを使用してリソースのプロパティを定義できます。プロパティは、リソースと同じディレクトリにある `.yml` ファイルで宣言できます。これらのファイルには `whatever_you_want.yml` という名前を付け、各ディレクトリ内のサブフォルダに任意にネストできます。

プロパティは、記述するリソースと同じパスに定義することを強くお勧めします。

:::info

#### schema.yml ファイル

以前のバージョンのドキュメントでは、これらのファイルは「schema.yml」ファイルと呼ばれていましたが、「schema」という言葉はデータベースに関する用語として他の意味にも使用され、これらのファイルは「schema.yml」という名前にしなければならないと思われていたため、この用語は廃止されました。

代わりに、これらのファイルは「properties.yml」ファイルと呼ばれるようになりました。（もちろん、ファイル名を「schema.yml」のままにしておくこともできます。）
:::

### 構成ではないプロパティはどれですか？

dbt では、`config()` ブロックと `dbt_project.yml` に加えて、`properties.yml` ファイルでもノード構成を定義できます。ただし、一部の特殊なプロパティは `.yml` ファイルでのみ定義でき、`config()` ブロックや `dbt_project.yml` ファイルでは設定できません。

以下の理由で特別なプロパティがあります。

- 固有の Jinja レンダリングコンテキストを持つ。
- 新しいプロジェクトリソースを作成する。
- 階層的な構成としては意味をなさない。
- まだ構成として再定義されていない古いプロパティである。

これらのプロパティは次のとおりです:

- [`columns`](/reference/resource-properties/columns)
- [`deprecation_date`](/reference/resource-properties/deprecation_date)
- [`description`](/reference/resource-properties/description)
- [`quote`](/reference/resource-properties/columns#quote)
- [`source` properties](/reference/source-properties) (for example, `loaded_at_field`, `freshness`)
- [`exposure` properties](/reference/exposure-properties) (for example, `type`, `maturity`)
  - ほとんどの公開プロパティは `properties.yml` ファイルで直接設定する必要がありますが、[`enabled`](/reference/resource-configs/enabled) 設定は `dbt_project.yml` ファイルの [プロジェクト レベル](/reference/exposure-properties#project-level-configs) で設定できることに注意してください。
- [`macro` properties](/reference/macro-properties) (for example, `arguments`)
- [`tests`](/reference/resource-properties/data-tests)
- [`versions`](/reference/resource-properties/versions)

import Example from '/snippets.ja/_configs-properties.md'  ;

<Example />
