---
title: "dbt environment コマンドについて"
sidebar_label: "environment"
id: dbt-environment
---

`dbt environment` コマンドを使用すると、dbt Cloud 環境を操作できます。このコマンドは次の目的で使用できます。

- ローカル構成の詳細（アカウント ID、アクティブ プロジェクト ID、デプロイメント環境など）の表示。
- dbt Cloud 構成の詳細（環境 ID、環境名、接続タイプなど）の表示。

このガイドでは、[dbt Cloud CLI](/docs/cloud/cloud-cli-installation) の `dbt environment` で使用できるすべてのコマンドとオプションを一覧表示しています。これらを使用するには、`dbt environment [command]` のようにコマンドまたはオプションを追加するか、短縮形 `dbt env [command]` を使用します。

### dbt environment show

`show` コマンド &mdash; ローカルおよび dbt Cloud の設定の詳細を表示します。dbt Cloud CLI でこのコマンドを実行するには、次のコマンドのいずれか（省略形を含む）を入力します。

```shell
dbt environment show
```
```shell
dbt env show
```

このコマンドは次の情報を返します:

```bash
❯ dbt env show
Local Configuration:
  Active account ID              185854
  Active project ID              271692
  Active host name               cloud.getdbt.com
  dbt_cloud.yml file path        /Users/cesar/.dbt/dbt_cloud.yml
  dbt_project.yml file path      /Users/cesar/git/cloud-cli-test-project/dbt_project.yml
  dbt Cloud CLI version          0.35.7
  OS info                        darwin arm64

Cloud Configuration:
  Account ID                     185854
  Project ID                     271692
  Project name                   Snowflake
  Environment ID                 243762
  Environment name               Development
  Defer environment ID           [N/A]
  dbt version                    1.6.0-latest
  Target name                    default
  Connection type                snowflake

Snowflake Connection Details:
  Account                        ska67070
  Warehouse                      DBT_TESTING_ALT
  Database                       DBT_TEST
  Schema                         CLOUD_CLI_TESTING
  Role                           SYSADMIN
  User                           dbt_cloud_user
  Client session keep alive      false 
```

dbt Cloud は秘密鍵を何も返さず、構成されていないフィールドには「NA」を返すことに注意してください。

### dbt environment flags

`dbt environment` コマンドでは、以下のフラグ（またはオプション）を使用します:

- `-h`、`--help` - コマンドラインインターフェースで特定のコマンドのヘルプドキュメントを表示します。

  ```shell 
  dbt environment [command] --help
  dbt environment [command] -h
  ```

  `--help` フラグは次の情報を返します:

  ```bash
    ❯ dbt help environment
    Interact with dbt environments

  Usage:
    dbt environment [command]

  Aliases:
    environment, env

  Available Commands:
    show        Show the working environment

  Flags:
    -h, --help   help for environment

  Use "dbt environment [command] --help" for more information about a command.
  ```

  たとえば、`show` コマンドのヘルプ ドキュメントを表示するには、ショートカットを含む次のコマンドのいずれかを入力します。

  ```shell
  dbt environment show --help
  dbt env show -h
  ```
