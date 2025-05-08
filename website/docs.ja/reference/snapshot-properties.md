---
title: Snapshot プロパティ
description: "dbt でのソース プロパティの使用については、このガイドをお読みください。"
---

<VersionBlock firstVersion="1.9">

dbt v1.9 以降では、snapshot は `snapshots/` ディレクトリ内の YAML ファイルで定義および構成されます（[`snapshot-paths` 設定](/reference/project-configs/snapshot-paths) で定義されています）。snapshot のプロパティはこれらの YAML ファイル内で宣言されるため、snapshot の構成とプロパティの両方を 1 か所で定義できます。

</VersionBlock>

<VersionBlock lastVersion="1.8">

Snapshots properties can be declared in `.yml` files in:
- your `snapshots/` directory (as defined by the [`snapshot-paths` config](/reference/project-configs/snapshot-paths)).
- your `models/` directory (as defined by the [`model-paths` config](/reference/project-configs/model-paths))

Note, in dbt v1.9 and later, snapshots are defined in an updated syntax using a YAML file within your `snapshots/` directory (as defined by the [`snapshot-paths` config](/reference/project-configs/snapshot-paths)). For faster and more efficient management, consider the updated snapshot YAML syntax, available now in [the dbt Cloud "Latest" release track](/docs/dbt-versions/cloud-release-tracks) and soon in [dbt Core v1.9](/docs/dbt-versions/core-upgrade/upgrading-to-v1.9).

</VersionBlock>

`snapshots/` ディレクトリに配置することをお勧めします。これらのファイルには `whatever_you_want.yml` という名前を付け、`snapshots/` または `models/` ディレクトリ内のサブフォルダに任意の深さでネストすることができます。

<VersionBlock firstVersion="1.9">

<File name='snapshots/<filename>.yml'>

```yml
version: 2

snapshots:
  - name: <snapshot name>
    [description](/reference/resource-properties/description): <markdown_string>
    [meta](/reference/resource-configs/meta): {<dictionary>}
    [docs](/reference/resource-configs/docs):
      show: true | false
      node_color: <color_id> # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
    [config](/reference/resource-properties/config):
      [<snapshot_config>](/reference/snapshot-configs): <config_value>
    [tests](/reference/resource-properties/data-tests):
      - <test>
      - ...
    columns:
      - name: <column name>
        [description](/reference/resource-properties/description): <markdown_string>
        [meta](/reference/resource-configs/meta): {<dictionary>}
        [quote](/reference/resource-properties/columns#quote): true | false
        [tags](/reference/resource-configs/tags): [<string>]
        [tests](/reference/resource-properties/data-tests):
          - <test>
          - ... # declare additional tests
      - ... # declare properties of additional columns

    - name: ... # declare properties of additional snapshots

```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

<File name='snapshots/<filename>.yml'>

```yml
version: 2

snapshots:
  - name: <snapshot name>
    [description](/reference/resource-properties/description): <markdown_string>
    [meta](/reference/resource-configs/meta): {<dictionary>}
    [docs](/reference/resource-configs/docs):
      show: true | false
      node_color: <color_id> # Use name (such as node_color: purple) or hex code with quotes (such as node_color: "#cd7f32")
    [config](/reference/resource-properties/config):
      [<snapshot_config>](/reference/snapshot-configs): <config_value>
    [tests](/reference/resource-properties/data-tests):
      - <test>
      - ...
    columns:
      - name: <column name>
        [description](/reference/resource-properties/description): <markdown_string>
        [meta](/reference/resource-configs/meta): {<dictionary>}
        [quote](/reference/resource-properties/columns#quote): true | false
        [tags](/reference/resource-configs/tags): [<string>]
        [tests](/reference/resource-properties/data-tests):
          - <test>
          - ... # declare additional tests
      - ... # declare properties of additional columns

    - name: ... # declare properties of additional snapshots

```
</File>
</VersionBlock>
