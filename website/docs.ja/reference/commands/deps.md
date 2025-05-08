---
title: "dbt deps コマンドについて"
sidebar_label: "deps"
id: "deps"
---

`dbt deps` は、`packages.yml` にリストされている依存関係の最新バージョンを Git から取得します。詳細については、[パッケージ管理](/docs/build/packages) を参照してください。

dbt は、該当する場合、dbt Hub にリストされているパッケージの最新バージョンを表示します。以下に例を示します。

> これは、git/local 経由でインストールされたパッケージには適用されません。

```yaml
packages:
  - package: dbt-labs/dbt_utils
    version: 0.7.1
  - package: brooklyn-data/dbt_artifacts
    version: 1.2.0
    install-prerelease: true
  - package: dbt-labs/codegen
    version: 0.4.0
  - package: calogica/dbt_expectations
    version: 0.4.1
  - git: https://github.com/dbt-labs/dbt-audit-helper.git
    revision: 0.4.0
  - git: "https://github.com/dbt-labs/dbt-labs-experimental-features" # git URL
    subdirectory: "materialized-views" # name of subdirectory containing `dbt_project.yml`
    revision: 0.0.1
  - package: dbt-labs/snowplow
    version: 0.13.0
```

```txt
Installing dbt-labs/dbt_utils@0.7.1
  Installed from version 0.7.1
  Up to date!
Installing brooklyn-data/dbt_artifacts@1.2.0
  Installed from version 1.2.0
Installing dbt-labs/codegen@0.4.0
  Installed from version 0.4.0
  Up to date!
Installing calogica/dbt_expectations@0.4.1
  Installed from version 0.4.1
  Up to date!
Installing https://github.com/dbt-labs/dbt-audit-helper.git@0.4.0
  Installed from revision 0.4.0
Installing https://github.com/dbt-labs/dbt-labs-experimental-features@0.0.1
  Installed from revision 0.0.1
   and subdirectory materialized-views
Installing dbt-labs/snowplow@0.13.0
  Installed from version 0.13.0
  Updated version available: 0.13.1
Installing calogica/dbt_date@0.4.0
  Installed from version 0.4.0
  Up to date!

Updates available for packages: ['tailsdotcom/dbt_artifacts', 'dbt-labs/snowplow']
Update your versions in packages.yml, then run dbt deps
```

## 予測可能なパッケージインストール

dbt v1.7 以降、dbt はプロジェクトのルートに `package-lock.yml` ファイルを生成します。このファイルは、`packages.yml` または `dependencies.yml` で指定されたすべての解決済みパッケージの正確なバージョン（コミット SHA を含む）を保存することで、一貫性があり予測可能なパッケージインストールを保証します。この一貫性は、開発環境と本番環境の安定性を維持し、潜在的なバグを含む新しいリリースによる予期せぬ問題を防ぐために不可欠です。

`dbt deps` を実行すると、dbt は `package-lock.yml` でロックされたバージョンに基づいてパッケージをインストールします。これらのロックされたバージョンを更新するには、明示的に `dbt deps --upgrade` を実行し、更新された `package-lock.yml` ファイルをコミットする必要があります。このファイルをバージョン管理に保存することで、すべての環境とすべての開発者にとって一貫性が保証されます。

### `package-lock.yml` の管理

`package-lock.yml` ファイルは最初に Git にコミットし、バージョンの変更やパッケージのアンインストールを行う場合にのみ更新してください。たとえば、パッケージのバージョンを更新するには `dbt deps --upgrade` を実行し、パッケージをインストールせずにパッケージ設定の変更に基づいてロックファイルを更新するには `dbt deps --lock` を実行します。

`package-lock.yml` の使用を完全に回避するには、プロジェクトの `.gitignore` に追加します。ただし、この方法ではビルドの予測可能性が損なわれます。この方法を選択する場合は、`packages` 設定でサードパーティ製パッケージのバージョンピンを追加することを強くお勧めします。

### `packages` 設定の変更を検出しています

`package-lock.yml` ファイルには、パッケージ設定の `sha1_hash` が含まれています。`packages.yml` を更新すると、dbt は変更を検出し、次回の `dbt deps` コマンド実行時に依存関係解決を再実行します。新しいパッケージをインストールせずにロックファイルを更新するには、`--lock` フラグを使用します。

```shell
dbt deps --lock
```

### パッケージの強制更新

`packages.yml` が変更されていない場合でも、すべてのパッケージを更新するには、`--upgrade` フラグを使用します:

```shell

dbt deps --upgrade

```

これは、内部的に管理されている Git パッケージの `main` ブランチから最新のコミットを取得する場合に特に便利です。

:::warning
慎重に管理しないと、パッケージのアップグレードを強制するとビルドの不整合が生じる可能性があります。
:::

### 特定のパッケージの追加

`dbt deps` コマンドを使用すると、パッケージ設定を直接追加または更新できるため、正確な構文を覚える必要がありません。

#### Hub パッケージ（デフォルト）

Hub パッケージはデフォルトのパッケージタイプであり、最も簡単にインストールできます。

```shell
dbt deps --add-package dbt-labs/dbt_utils@1.0.0

# with semantic version range
dbt deps --add-package dbt-labs/snowplow@">=0.7.0,<0.8.0"
```

#### Hub 以外のパッケージ

インストールするパッケージの種類を指定するには、`--source` フラグを使用します。

```shell

# Git package
dbt deps --add-package https://github.com/fivetran/dbt_amplitude@v0.3.0 --source git

# Local package
dbt deps --add-package /opt/dbt/redshift --source local
```
