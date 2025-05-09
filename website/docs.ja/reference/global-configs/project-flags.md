---
title: "プロジェクトフラグ"
id: "project-flags"
sidebar: "プロジェクトフラグ"
---

<File name='dbt_project.yml'>

```yaml

flags:
  <global_config>: <value>

```

</File>

[`dbt_project.yml`](/reference/dbt_project.yml) で設定可能なグローバル設定を確認するには、[すべてのフラグの一覧](/reference/global-configs/about-global-configs#available-flags)を参照してください。

`flags` ディクショナリは、[動作の変更](/reference/global-configs/behavior-changes) をオプトアウトできる唯一の場所ですが、従来の動作は引き続きサポートされます。

<VersionBlock lastVersion="1.7">

:::warning Deprecated functionality
In older versions of dbt, custom default values of flags (global configs) were set in `profiles.yml`. Starting in v1.8, these configs are set in `dbt_project.yml` instead.
:::

For most global configurations, you can set "user profile" configurations in the `config:` block of `profiles.yml`. This style of configuration sets default values for all projects using this profile directory &mdash; usually, all projects running on your local machine.

<File name='profiles.yml'>

```yaml

config:
  <THIS-CONFIG>: true

```

</File>

</VersionBlock>

<VersionBlock lastVersion="1.7">

The exception: Some global configurations are actually set in `dbt_project.yml`, instead of `profiles.yml`, because they control where dbt places logs and artifacts. Those file paths are always relative to the location of `dbt_project.yml`. For more details, refer to [Log and target paths](/reference/global-configs/logs#log-and-target-paths).

</VersionBlock>
