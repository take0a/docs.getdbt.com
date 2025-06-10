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
