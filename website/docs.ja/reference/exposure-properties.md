---
title: Exposure プロパティ
description: "dbt の exposure プロパティを理解するには、このガイドをお読みください。"
---

## 関連ドキュメント

- [エクスポージャーの使用](/docs/build/exposures)
- [リソースプロパティの宣言](/reference/configs-and-properties)

## 概要

import PropsCallout from '/snippets/_config-prop-callout.md';

エクスポージャーは、`exposures:` キーの下にネストされた `properties.yml` ファイルで定義されます。`sources` または `models` も定義している YAML ファイルで `exposures` を定義できます。<PropsCallout title={frontMatter.title}/> <br />

ほとんどのエクスポージャープロパティはこれらの YAML ファイルで直接設定する必要がありますが、[`enabled`](/reference/resource-configs/enabled) 設定は [プロジェクトレベル](#project-level-configs) の `dbt_project.yml` ファイルで設定できます。

これらのファイルには `whatever_you_want.yml` という名前を付け、`models/` ディレクトリ内の任意の深さのサブフォルダにネストできます。

エクスポージャー名には、文字、数字、アンダースコアのみを使用してください（スペースや特殊文字は使用できません）。タイトルの大文字小文字、スペース、特殊文字を含む、人間にとって分かりやすい短い名前を作成するには、`label` プロパティを使用してください。

<File name='models/<filename>.yml'>

```yml
version: 2

exposures:
  - name: <string_with_underscores>
    [description](/reference/resource-properties/description): <markdown_string>
    type: {dashboard, notebook, analysis, ml, application}
    url: <string>
    maturity: {high, medium, low}  # Indicates level of confidence or stability in the exposure
    [enabled](/reference/resource-configs/enabled): true | false
    [tags](/reference/resource-configs/tags): [<string>]
    [meta](/reference/resource-configs/meta): {<dictionary>}
    owner:
      name: <string>
      email: <string>
    
    depends_on:
      - ref('model')
      - ref('seed')
      - source('name', 'table')
      - metric('metric_name')
      
    label: "Human-Friendly Name for this Exposure!"
    [config](/reference/resource-properties/config):
      enabled: true | false

  - name: ... # declare properties of additional exposures
```
</File>

## Example

<File name='models/jaffle/exposures.yml'>

```yaml
version: 2

exposures:

  - name: weekly_jaffle_metrics
    label: Jaffles by the Week              # optional
    type: dashboard                         # required
    maturity: high                          # optional
    url: https://bi.tool/dashboards/1       # optional
    description: >                          # optional
      Did someone say "exponential growth"?

    depends_on:                             # expected
      - ref('fct_orders')
      - ref('dim_customers')
      - source('gsheets', 'goals')
      - metric('count_orders')

    owner:
      name: Callum McData
      email: data@jaffleshop.com


      
  - name: jaffle_recommender
    maturity: medium
    type: ml
    url: https://jupyter.org/mycoolalg
    description: >
      Deep learning to power personalized "Discover Sandwiches Weekly"
    
    depends_on:
      - ref('fct_orders')
      
    owner:
      name: Data Science Drew
      email: data@jaffleshop.com

      
  - name: jaffle_wrapped
    type: application
    description: Tell users about their favorite jaffles of the year
    depends_on: [ ref('fct_orders') ]
    owner: { email: summer-intern@jaffleshop.com }
```

</File>

#### プロジェクトレベルの設定

`dbt_project.yml` ファイル内の `exposures:` キーに `+` プレフィックスを付けて、エクスポージャーに関するプロジェクトレベルの設定を定義できます。現在、[`enabled` 設定](/reference/resource-configs/enabled) のみがサポートされています。

<File name="dbt_project.yml">

```yml
name: 'project_name'

# rest of dbt_project.yml

exposures:
  +enabled: true
```

</File>
