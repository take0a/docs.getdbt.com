---
title: Seed 構成
description: "dbt での seed 構成の使用については、このガイドをお読みください。"
meta:
  resource_type: Seeds
---

import ConfigResource from '/snippets.ja/_config-description-resource.md';
import ConfigGeneral from '/snippets.ja/_config-description-general.md';


## 利用可能な構成

###  seed 固有の構成

<ConfigResource meta={frontMatter.meta} />

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Property file', value: 'property-yaml', },
  ]
}>
<TabItem value="project-yaml">

<File name='dbt_project.yml'>

```yml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[quote_columns](/reference/resource-configs/quote_columns): true | false
    [+](/reference/resource-configs/plus-prefix)[column_types](/reference/resource-configs/column_types): {column_name: datatype}
    [+](/reference/resource-configs/plus-prefix)[delimiter](/reference/resource-configs/delimiter): <string>

```

</File>

</TabItem>


<TabItem value="property-yaml">

<File name='seeds/properties.yml'>

```yaml
version: 2

seeds:
  - name: [<seed-name>]
    config:
      [quote_columns](/reference/resource-configs/quote_columns): true | false
      [column_types](/reference/resource-configs/column_types): {column_name: datatype}
      [delimiter](/reference/resource-configs/grants): <string>

```

</File>

</TabItem>

</Tabs>

### 一般的な構成

<ConfigGeneral />

<Tabs
  groupId="config-languages"
  defaultValue="project-yaml"
  values={[
    { label: 'Project file', value: 'project-yaml', },
    { label: 'Property file', value: 'property-yaml', },
  ]
}>

<TabItem value="project-yaml">

<File name='dbt_project.yml'>

<VersionBlock lastVersion="1.8">

```yaml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[pre-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[post-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[database](/reference/resource-configs/database): <string>
    [+](/reference/resource-configs/plus-prefix)[schema](/reference/resource-properties/schema): <string>
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[persist_docs](/reference/resource-configs/persist_docs): <dict>
    [+](/reference/resource-configs/plus-prefix)[full_refresh](/reference/resource-configs/full_refresh): <boolean>
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[grants](/reference/resource-configs/grants): {<dictionary>}

```
</VersionBlock>

<VersionBlock firstVersion="1.9">

```yaml
seeds:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[tags](/reference/resource-configs/tags): <string> | [<string>]
    [+](/reference/resource-configs/plus-prefix)[pre-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[post-hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
    [+](/reference/resource-configs/plus-prefix)[database](/reference/resource-configs/database): <string>
    [+](/reference/resource-configs/plus-prefix)[schema](/reference/resource-properties/schema): <string>
    [+](/reference/resource-configs/plus-prefix)[alias](/reference/resource-configs/alias): <string>
    [+](/reference/resource-configs/plus-prefix)[persist_docs](/reference/resource-configs/persist_docs): <dict>
    [+](/reference/resource-configs/plus-prefix)[full_refresh](/reference/resource-configs/full_refresh): <boolean>
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[grants](/reference/resource-configs/grants): {<dictionary>}
    [+](/reference/resource-configs/plus-prefix)[event_time](/reference/resource-configs/event-time): my_time_field

```
</VersionBlock>
</File>

</TabItem>


<TabItem value="property-yaml">

<File name='seeds/properties.yml'>

<VersionBlock firstVersion="1.9">

```yaml
version: 2

seeds:
  - name: [<seed-name>]
    config:
      [enabled](/reference/resource-configs/enabled): true | false
      [tags](/reference/resource-configs/tags): <string> | [<string>]
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [database](/reference/resource-configs/database): <string>
      [schema](/reference/resource-properties/schema): <string>
      [alias](/reference/resource-configs/alias): <string>
      [persist_docs](/reference/resource-configs/persist_docs): <dict>
      [full_refresh](/reference/resource-configs/full_refresh): <boolean>
      [meta](/reference/resource-configs/meta): {<dictionary>}
      [grants](/reference/resource-configs/grants): {<dictionary>}
      [event_time](/reference/resource-configs/event-time): my_time_field

```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
version: 2

seeds:
  - name: [<seed-name>]
    config:
      [enabled](/reference/resource-configs/enabled): true | false
      [tags](/reference/resource-configs/tags): <string> | [<string>]
      [pre_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [post_hook](/reference/resource-configs/pre-hook-post-hook): <sql-statement> | [<sql-statement>]
      [database](/reference/resource-configs/database): <string>
      [schema](/reference/resource-properties/schema): <string>
      [alias](/reference/resource-configs/alias): <string>
      [persist_docs](/reference/resource-configs/persist_docs): <dict>
      [full_refresh](/reference/resource-configs/full_refresh): <boolean>
      [meta](/reference/resource-configs/meta): {<dictionary>}
      [grants](/reference/resource-configs/grants): {<dictionary>}
```

</VersionBlock>
</File>

</TabItem>
</Tabs>

## seed の設定

seed は、`dbt_project.yml` または個々の seed の YAML プロパティ内の YAML ファイルからのみ設定できます。CSV ファイル内から seed を設定することはできません。

モデル設定と同様に、 seed 設定は階層的に適用されます。つまり、`marketing` サブディレクトリに適用された設定は、`jaffle_shop` プロジェクト全体に適用された設定よりも優先されます。また、特定の seed のプロパティで定義された設定は、`dbt_project.yml` で定義された設定をオーバーライドします。

### 例

#### すべての seed に `schema` 設定を適用する

インストール済みの [パッケージ](/docs/build/packages) 内の seed も含め、すべての seed に設定を適用するには、設定を `seeds` キーの直下にネストします。

<File name='dbt_project.yml'>

```yml

seeds:
  +schema: seed_data
```

</File>


#### プロジェクト内のすべての seed に `schema` 設定を適用する

プロジェクト内のすべての seed にのみ設定を適用するには（つまり、インストール済みパッケージ内の seed は_除外_）、リソースパスの一部として [プロジェクト名](/reference/project-configs/name.md) を指定します。

`jaffle_shop` というプロジェクトの場合：

<File name='dbt_project.yml'>

```yml

seeds:
  jaffle_shop:
    +schema: seed_data
```

</File>

同様に、インストールされたパッケージの名前を使用して、そのパッケージ内の seed を設定することもできます。

#### `schema` 構成を 1 つの seed にのみ適用する

構成を 1 つの seed にのみ適用するには、完全なリソースパス（プロジェクト名とサブディレクトリを含む）を指定します。

<File name='seeds/marketing/properties.yml'>

```yml
version: 2

seeds:
  - name: utm_parameters
    config:
      schema: seed_data
```

</File>

dbtの古いバージョンでは、`dbt_project.yml`で設定を定義し、リソースの完全なパス（プロジェクト名とサブディレクトリを含む）を含める必要があります。`jaffle_shop`というプロジェクトで、 seed ファイルが`seeds/marketing/utm_parameters.csv`にある場合、以下のようになります。

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop:
    marketing:
      utm_parameters:
        +schema: seed_data
```

</File>


## seed 設定の例

以下のプロジェクトに有効な seed 設定は次のとおりです。
* `name: jaffle_shop`
* `seeds/country_codes.csv` にある seed ファイル、および
* `seeds/marketing/utm_parameters.csv` にある seed ファイル


<File name='dbt_project.yml'>

```yml
name: jaffle_shop
...
seeds:
  jaffle_shop:
    +enabled: true
    +schema: seed_data
    # This configures seeds/country_codes.csv
    country_codes:
      # Override column types
      +column_types:
        country_code: varchar(2)
        country_name: varchar(32)
    marketing:
      +schema: marketing # this will take precedence
```

</File>
