---
title: "dbt init コマンドについて"
sidebar_label: "init"
id: "init"
---

`dbt init` は、dbt Core の使用を開始するのに役立ちます。

## 新規プロジェクト

このツールを初めて使用する場合は、以下の手順が表示されます。
- プロジェクト名の入力を求められます。
- 使用しているデータベースアダプタ（または[サポートされているデータプラットフォーム](/docs/supported-data-platforms)）の入力を求められます。
- dbt がデータベースに接続するために必要な情報（アカウント、ユーザー、パスワードなど）の入力を求められます。

その後、以下の手順が表示されます。
- プロジェクト名とサンプルファイルを含む新しいフォルダが作成されます。dbt を使い始めるのに十分な情報です。
- ローカルマシンに接続プロファイルが作成されます。デフォルトの場所は `~/.dbt/profiles.yml` です。詳しくは、[プロファイルの設定](/docs/core/connect-data-platform/connection-profiles) をご覧ください。

`dbt init` を使用してプロジェクトを初期化する際、`--profile` フラグを指定して、`profile:` キーとして既存の `profiles.yml` を指定します。新しいプロファイルを作成する必要はありません。たとえば、`dbt init --profile profile_name` のように指定します。

`profiles.yml` にプロファイルが存在しない場合、または既存のプロジェクト内でコマンドを実行すると、エラーが発生します。


## 既存のプロジェクト

既存の dbt プロジェクトをクローンまたはダウンロードした場合でも、`dbt init` を使用すると接続プロファイルを設定でき、すぐに作業を開始できます。上記のように接続情報の入力を求められ、プロジェクトの `profile` 名を使用してローカルの `profiles.yml` にプロファイルが追加されます。ファイルがまだ存在しない場合は作成されます。


## profile_template.yml

`dbt init` は、`profile_template.yml` というファイルを検索することで接続情報の入力を求めます。このファイルは、以下の 2 つの場所で検索されます。

- **アダプタプラグイン:** 最低限必要な Postgres プロファイルは何ですか？各フィールドのタイプとデフォルト値は何ですか？この情報は、[`dbt/include/postgres/profile_template.yml`](https://github.com/dbt-labs/dbt-postgres/blob/main/dbt/include/postgres/profile_template.yml) というファイルに保存されています。アダプタプラグインのメンテナーの方は、プラグインにも `profile_template.yml` を追加することを強くお勧めします。詳細については、[アダプタのビルド、テスト、ドキュメント化、およびプロモート](/guides/adapter-creation) ガイドを参照してください。

- **既存プロジェクト:** 既存プロジェクトのメンテナーで、新規ユーザーがデータベースに迅速かつ簡単に接続できるようにしたい場合は、プロジェクトのルートに `dbt_project.yml` と並んで、独自のカスタム `profile_template.yml` を含めることができます。共通の接続属性については `fixed` で値を設定し、ユーザー固有の属性は `prompts` に残し、必要に応じてカスタムヒントとデフォルトを設定できます。

<File name='profile_template.yml'>

```yml
fixed:
  account: abc123
  authenticator: externalbrowser
  database: analytics
  role: transformer
  type: snowflake
  warehouse: transforming
prompts:
  target:
    type: string
    hint: your desired target name
  user:
    type: string
    hint: yourname@jaffleshop.com
  schema:
    type: string
    hint: usually dbt_<yourname>
  threads:
    hint: "your favorite number, 1-10"
    type: int
    default: 8
```

</File>


```
$ dbt init
Running with dbt=1.0.0
Setting up your profile.
user (yourname@jaffleshop.com): summerintern@jaffleshop.com
schema (usually dbt_<yourname>): dbt_summerintern
threads (your favorite number, 1-10) [8]: 6
Profile internal-snowflake written to /Users/intern/.dbt/profiles.yml using project's profile_template.yml and your supplied values. Run 'dbt debug' to validate the connection.
```
