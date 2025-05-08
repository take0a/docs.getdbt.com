---
title: Source 構成
description: "dbt で source 構成を使用する方法を学習します。"
id: source-configs
---

import ConfigGeneral from '/snippets.ja/_config-description-general.md';

## 利用可能な構成

<VersionBlock lastVersion="1.8">

Sources supports [`enabled`](/reference/resource-configs/enabled) and [`meta`](/reference/resource-configs/meta).

</VersionBlock>

<VersionBlock firstVersion="1.9">

ソース構成は、[`enabled`](/reference/resource-configs/enabled)、[`event_time`](/reference/resource-configs/event-time)、[`meta`](/reference/resource-configs/meta) をサポートします。

</VersionBlock>

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

<VersionBlock firstVersion="1.9">

```yaml
sources:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[event_time](/reference/resource-configs/event-time): my_time_field
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta):
      key: value

```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
sources:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[enabled](/reference/resource-configs/enabled): true | false
    [+](/reference/resource-configs/plus-prefix)[meta](/reference/resource-configs/meta):
      key: value
```
</VersionBlock>

</File>

</TabItem>


<TabItem value="property-yaml">

<File name='models/properties.yml'>

<VersionBlock firstVersion="1.9">

```yaml
version: 2

sources:
  - name: [<source-name>]
    [config](/reference/resource-properties/config):
      [enabled](/reference/resource-configs/enabled): true | false
      [event_time](/reference/resource-configs/event-time): my_time_field
      [meta](/reference/resource-configs/meta): {<dictionary>}

    tables:
      - name: [<source-table-name>]
        [config](/reference/resource-properties/config):
          [enabled](/reference/resource-configs/enabled): true | false
          [event_time](/reference/resource-configs/event-time): my_time_field
          [meta](/reference/resource-configs/meta): {<dictionary>}

```
</VersionBlock>

<VersionBlock lastVersion="1.8">

```yaml
version: 2

sources:
  - name: [<source-name>]
    [config](/reference/resource-properties/config):
      [enabled](/reference/resource-configs/enabled): true | false
      [meta](/reference/resource-configs/meta): {<dictionary>}
    tables:
      - name: [<source-table-name>]
        [config](/reference/resource-properties/config):
          [enabled](/reference/resource-configs/enabled): true | false
          [meta](/reference/resource-configs/meta): {<dictionary>}

```
</VersionBlock>

</File>

</TabItem>

</Tabs>

## ソースの設定

ソースは、`.yml` 定義内の `config:` ブロック、または `dbt_project.yml` ファイルの `sources:` キーで設定できます。この設定は、[パッケージ](/docs/build/packages) からインポートされたソースを設定する場合に最も便利です。

パッケージからインポートされたソースを無効にすると、ドキュメントにソースがレンダリングされないようにしたり、パッケージからインポートされたソーステーブルに対して [ソースの鮮度チェック](/docs/build/sources#source-data-freshness) が実行されないようにしたりできます。

- **注**: サブフォルダ内の YAML ファイルにネストされたソーステーブルを無効にするには、その YAML ファイルへのパスにサブフォルダを指定し、`dbt_project.yml` ファイルにソース名とテーブル名を指定する必要があります。<br /><br />
次の例は、サブフォルダ内の YAML ファイルにネストされたソーステーブルを無効にする方法を示しています。

  <File name='dbt_project.yml'>

  <VersionBlock firstVersion="1.9">

  ```yaml
  sources:
    your_project_name:
      subdirectory_name:
        source_name:
          source_table_name:
            +enabled: false
            +event_time: my_time_field
  ```

  </VersionBlock>

  <VersionBlock lastVersion="1.8">
    ```yaml
  sources:
    your_project_name:
      subdirectory_name:
        source_name:
          source_table_name:
            +enabled: false
  ```
  </VersionBlock>
  </File>


### 例

以下の例は、dbt プロジェクトでソースを構成する方法を示しています。

&mdash; [パッケージからインポートされたすべてのソースを無効にする](#disable-all-sources-imported-from-a-package) <br />
&mdash; [条件付きで単一のソースを有効にする](#conditionally-enable-a-single-source) <br />
&mdash; [パッケージから単一のソースを無効にする](#disable-a-single-source-from-a-package) <br />
&mdash; [`event_time` を使用してソースを構成する](#configure-a-source-with-an-event_time) <br />
&mdash; [ソースにメタを構成する](#configure-meta-to-a-source) <br />

#### パッケージからインポートされたすべてのソースを無効にする {#disable-all-sources-imported-from-a-package}

[パッケージ](/docs/build/packages) に含まれるすべてのソースに設定を適用するには、リソースパスの一部として、`sources:` 設定の [プロジェクト名](/reference/project-configs/name.md) の下に設定を記述します。

<File name='dbt_project.yml'>

```yml
sources:
  events:
    +enabled: false
```

</File>


#### 条件付きで単一のソースを有効にする {#conditionally-enable-a-single-source}

ソースを定義するときに、インライン `config` プロパティを使用して、ソース全体または特定のソース テーブルを無効にすることができます:

<File name='models/sources.yml'>

```yml
version: 2

sources:
  - name: my_source
    config:
      enabled: true
    tables:
      - name: my_source_table  # enabled
      - name: ignore_this_one  # not enabled
        config:
          enabled: false
```

</File>

特定のソース テーブルを構成し、その構成への入力として [変数](/reference/dbt-jinja-functions/var) を使用できます:
 
<File name='models/sources.yml'>

```yml
version: 2

sources:
  - name: my_source
    tables:
      - name: my_source_table
        config:
          enabled: "{{ var('my_source_table_enabled', false) }}"
```

</File>

#### パッケージから単一のソースを無効にする {#disable-a-single-source-from-a-package}

別のパッケージの特定のソースを無効にするには、設定のリソースパスをパッケージ名とソース名の両方で修飾します。この例では、`events` パッケージの `clickstream` ソースを無効にします。

<File name='dbt_project.yml'>

```yml
sources:
  events:
    clickstream:
      +enabled: false
```

</File>

同様に、リソース パスをパッケージ名、ソース名、テーブル名で修飾することで、ソースから特定のテーブルを無効にすることもできます。

<File name='dbt_project.yml'>

```yml
sources:
  events:
    clickstream:
      pageviews:
        +enabled: false
```

</File>


#### `event_time` を使用してソースを構成する {#configure-a-source-with-an-event_time}

<VersionBlock lastVersion="1.8">

Configuring an [`event_time`](/reference/resource-configs/event-time) for a source is only available in [the dbt Cloud "Latest" release track](/docs/dbt-versions/cloud-release-tracks) or dbt Core versions 1.9 and later.

</VersionBlock>

<VersionBlock firstVersion="1.9">

`event_time` を含むソースを設定するには、ソース設定で `event_time` フィールドを指定します。このフィールドは、読み込み日時などの情報ではなく、イベントの実際のタイムスタンプを表すために使用されます。

例えば、`events` ソースに `clickstream` というソーステーブルがある場合、次のように `event_timestamp` 列で各イベントのタイムスタンプを使用できます。

<File name='dbt_project.yml'>

```yaml
sources:
  events:
    clickstream:
      +event_time: event_timestamp
```
</File>

この例では、`event_time` は、各クリックストリーム イベントが発生した正確な時刻を持つ `event_timestamp` に設定されています。
これは、[増分マイクロバッチ戦略](/docs/build/incremental-microbatch)に必要なだけでなく、[CI と本番環境](/docs/deploy/advanced-ci#speeding-up-comparisons)間でデータを比較する場合、dbt は `event_timestamp` を使用して、このイベントベースの時間枠でデータをフィルタリングおよび照合し、重複する時間枠のみが比較されるようにします。

</VersionBlock>

#### ソースにメタを構成する {#configure-meta-to-a-source}

`meta` フィールドを使用して、ソースにメタデータ情報を割り当てます。これは、追加のコンテキスト、ドキュメント、ログなどの追跡に役立ちます。

例えば、`clickstream` ソースに `meta` 情報を追加して、データソースシステムに関する情報を含めることができます。

<File name='dbt_project.yml'>

```yaml
sources:
  events:
    clickstream:
      +meta:
        source_system: "Google analytics"
        data_owner: "marketing_team"
```
</File>

## ソース構成の例

以下は、以下のプロジェクトに有効なソース構成です。
* `name: jaffle_shop`
* 複数のソーステーブルを含む `events` というパッケージ


<File name='dbt_project.yml'>

```yml
name: jaffle_shop
config-version: 2
...
sources:
  # project names
  jaffle_shop:
    +enabled: true

  events:
    # source names
    clickstream:
      # table names
      pageviews:
        +enabled: false
      link_clicks:
        +enabled: true
```

</File>
