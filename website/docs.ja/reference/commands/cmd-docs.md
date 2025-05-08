---
title: "dbt docs コマンドについて"
description: "dbt プロジェクトのドキュメントを生成して提供します。"
sidebar_label: "docs"
id: "cmd-docs"
---

`dbt docs` には、`generate` と `serve` という 2 つのサブコマンドがサポートされています。

### dbt docs generate

このコマンドは、プロジェクトのドキュメントウェブサイトを生成するために以下の処理を行います。

1. ウェブサイトの `index.html` ファイルを `target/` ディレクトリにコピーします。
2. プロジェクト内のリソースをコンパイルし、その `compiled_code` が [`manifest.json`](/reference/artifacts/manifest-json) に含まれるようにします。
3. データベースのメタデータに対してクエリを実行し、[`catalog.json`](/reference/artifacts/catalog-json) ファイルを生成します。このファイルには、プロジェクト内のモデルによって生成されたテーブルと <Term id="view">ビュー</Term> に関するメタデータが含まれています。

**例**：

```
dbt docs generate
```

`catalog.json` に含まれるノードを制限するには、`--select` 引数を使用します。このフラグが指定されている場合、ステップ (3) は選択されたノードのみに制限されます。その他のノードはすべて除外されます。ステップ (2) には影響しません。

**例**：

```shell
dbt docs generate --select +orders
```

再コンパイルをスキップするには、`--no-compile` 引数を使用します。このフラグが指定されると、`dbt docs generate` は上記のステップ (2) をスキップします。

**例**：

```
dbt docs generate --no-compile
```

`catalog.json` にデータを入力するデータベースクエリの実行をスキップするには、`--empty-catalog` 引数を使用します。このフラグが指定されると、`dbt docs generate` は上記の手順 (3) をスキップします。

これは、データベースメタデータ（各テーブルの列の完全なセットとそれらのテーブルに関する統計情報）から取得された情報がドキュメントに含まれなくなるため、本番環境では推奨されません。開発環境では、プロジェクト内で定義されたリネージやその他の情報を視覚化したいだけであれば、`docs generate` を高速化できます。dbt Cloud でドキュメントを作成する方法については、[dbt Cloud でドキュメントを作成する](/docs/collaborate/build-and-view-your-docs) を参照してください。

**例**：

```
dbt docs generate --empty-catalog
```

**例**：

`--static` フラグを使用すると、ドキュメントをクラウドストレージプロバイダーでホスティングするための静的ページとして生成できます。`catalog.json` ファイルと `manifest.json` ファイルが `index.html` ファイルに挿入され、メールやファイル共有アプリで簡単に共有できる単一のページが作成されます。

```
dbt docs generate --static
```

### dbt docs serve

このコマンドは、ポート 8080 でウェブサーバーを起動し、ドキュメントをローカルで提供して、デフォルトのブラウザでドキュメント サイトを開きます。ウェブサーバーのルートは `target/` ディレクトリです。`generate` コマンドは、`serve` コマンドが依存する [カタログ メタデータ アーティファクト](/reference/artifacts/catalog-json) を生成するため、`dbt docs generate` を `dbt docs serve` の前に必ず実行してください。カタログが見つからない場合は、エラー メッセージが表示されます。

[dbt Cloud CLI](/docs/cloud/cloud-cli-installation) または [dbt Core](/docs/core/installation-overview) を使用してローカルで開発している場合は、`dbt docs serve` コマンドを使用してください。[dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) はこのコマンドをサポートしていません。

**使用方法:**

<VersionBlock lastVersion="1.8.1">
```
dbt docs serve [--profiles-dir PROFILES_DIR]
               [--profile PROFILE] [--target TARGET]
               [--port PORT]
               [--no-browser]
```
</VersionBlock>
<VersionBlock firstVersion="1.8.2">
```
dbt docs serve [--profiles-dir PROFILES_DIR]
               [--profile PROFILE] [--target TARGET]
               [--host HOST]
               [--port PORT]
               [--no-browser]
```
</VersionBlock>

`--port` フラグを使用して別のポートを指定することもできます。

**例**：

```
dbt docs serve --port 8001
```

<VersionBlock firstVersion="1.8.2">

`--host` フラグを使用して別のホストを指定することもできます。

**例**：

```shell
dbt docs serve --host ""
```

1.8.1 以降、デフォルトのホストは `127.0.0.1` です。バージョン 1.8.0 以前では、デフォルトのホストは `""` でした。
</VersionBlock>
