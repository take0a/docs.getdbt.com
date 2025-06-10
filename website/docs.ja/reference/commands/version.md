---
title: "dbt --version について"
sidebar_label: "version"
id: "version"
---

`--version` コマンドラインフラグは、現在インストールされている <Constant name="core" />  または <Constant name="cloud_cli" /> のバージョンに関する情報を返します。このフラグは、他の <Constant name="cloud" /> ランタイム（IDE やスケジュールされた実行など）で dbt を呼び出す場合にはサポートされません。

- **<Constant name="core" /> ** &mdash; インストールされている dbt-core のバージョンと、インストールされているすべてのアダプタのバージョンを返します。
- **<Constant name="cloud_cli" />** &mdash; [<Constant name="cloud_cli" />](/docs/cloud/cloud-cli-installation) のバージョンを返します。その他の `dbt_version` 値の場合は、<Constant name="cloud" /> 内の dbt ランタイムの最新バージョンを返します。


## バージョニング

<Constant name="core" />  のリリース バージョン管理の詳細については、[<Constant name="core" />  におけるセマンティック バージョニングの仕組み](/docs/dbt-versions/core#how-dbt-core-uses-semantic-versioning) を参照してください。

dbt の継続的なアップデートを提供する [<Constant name="cloud" /> リリース トラック](/docs/dbt-versions/cloud-release-tracks) を使用する場合、`dbt_version` は <Constant name="cloud" /> における dbt のリリース バージョンを表します。これもセマンティック バージョニングのガイドラインに従い、`YYYY.M.D+<サフィックス>` 形式を使用します。年、月、日は、バージョンがビルドされた日付を表します（例: `2024.10.8+996c6a8`）。サフィックスは、各ビルドに固有の識別情報を追加します。

## 使用例

<Constant name="core" />  の例:

<File name='dbt Core'>

```text
$ dbt --version
Core:
  - installed: 1.7.6
  - latest:    1.7.6 - Up to date!
Plugins:
  - snowflake: 1.7.1 - Up to date!
```

</File>

<Constant name="cloud_cli" /> の例:

<File name='dbt Cloud CLI'>

```text
$ dbt --version
dbt Cloud CLI - 0.35.7 (fae78a6f5f6f2d7dff3cab3305fe7f99bd2a36f3 2024-01-18T22:34:52Z)
```

</File>
