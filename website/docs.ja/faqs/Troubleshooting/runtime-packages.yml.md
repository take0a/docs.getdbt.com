---
title: パッケージでランタイム エラーが発生するのはなぜですか?
description: "packages.yml ファイル内の dbt_utils パッケージを更新します。"
sidebar_label: 'packages.yml ファイルの実行時エラー'
id: runtime-packages.yml

---

packages.yml フォルダーで以下のランタイム エラーが表示される場合は、現在の dbt Cloud バージョンと互換性のない古いバージョンの dbt_utils パッケージが原因である可能性があります。

```shell
Running with dbt=xxx
Runtime Error
  Failed to read package: Runtime Error
    Invalid config version: 1, expected 2  
  Error encountered in dbt_utils/dbt_project.yml
  ```

packages.yml 内の dbt_utils パッケージの古いバージョンを [dbt hub](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) にある最新バージョンに更新してみてください。

```shell
packages:
- package: dbt-labs/dbt_utils

version: xxx
```

上記の回避策を試してもこの現象が引き続き発生する場合は、support@getdbt.com のサポート チームまでご連絡ください。喜んでお手伝いいたします。
