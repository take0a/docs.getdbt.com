---
title: "dbt clean コマンドについて"
sidebar_label: "clean"
id: "clean"
---

`dbt clean` は、`dbt_project.yml` ファイルの [`clean-targets`](/reference/project-configs/clean-targets) リスト内で指定されたパスを削除するユーティリティ関数です。他の dbt コマンドの実行中に生成された不要なファイルやディレクトリを削除することで、プロジェクトをクリーンな状態に保ちます。

## 使用例

```
dbt clean
```

## サポートされているフラグ

このセクションでは、以下のフラグについて簡単に説明します。

- [`--clean-project-files-only`](#--clean-project-files-only) (デフォルト)
- [`--no-clean-project-files-only`](#--no-clean-project-files-only)

ターミナルで `dbt clean` コマンドでサポートされているすべてのフラグのリストを表示するには、`--help` フラグを使用します。これにより、使用可能なフラグに関する詳細情報（説明や使用方法など）が表示されます。

```shell
dbt clean --help
```

### --clean-project-files-only

デフォルトでは、dbt は `clean-targets` で指定されたプロジェクト ディレクトリ内のすべてのパスを削除します。

:::note
dbt プロジェクト外のパスを使用しないでください。そうしないと、エラーが表示されます。
:::
  

#### 使用例

```shell
dbt clean --clean-project-files-only
```

### --no-clean-project-files-only

現在の dbt プロジェクト外のパスも含め、`dbt_project.yml` の `clean-targets` リストで指定されたすべてのパスを削除します。

```shell
dbt clean --no-clean-project-files-only
```

## リモートファイルシステムでのdbt clean

複雑な権限の問題や、修正権限がないままリモートファイルシステムの重要な部分が削除される可能性を回避するため、このコマンドはdbt Cloud IDEを動かすRPCサーバーとのインターフェースでは機能しません。dbt Cloud内で作業する場合、`dbt deps`コマンドはパッケージを自動的にインストールする前にクリーンアップを実行します。`target`フォルダは、必要に応じてサイドバーのファイルツリーから手動で削除できます。
