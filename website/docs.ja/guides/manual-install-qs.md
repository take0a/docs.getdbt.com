---
title: "手動インストールによる dbt Core のクイックスタート"
id: manual-install
description: "Connecting your warehouse to dbt Core using the CLI."
level: 'Beginner'
platform: 'dbt-core'
icon: 'fa-light fa-square-terminal'
tags: ['dbt Core','Quickstart']
hide_table_of_contents: true
---

<div style={{maxWidth: '900px'}}>

## 導入

dbt Core を使用して dbt を操作する場合、コード エディターを使用してローカルでファイルを編集し、コマンド ライン インターフェース (CLI) を使用してプロジェクトを実行します。

Web ベースの dbt 統合開発環境 (IDE) を使用してファイルを編集し、プロジェクトを実行する場合は、[dbt Cloud クイックスタート](/guides) を参照してください。また、[dbt Cloud CLI](/docs/cloud/cloud-cli-installation) (dbt Cloud を利用したコマンド ライン) を使用して dbt コマンドを開発および実行することもできます。

### 前提条件

* dbt Core を使用するには、ターミナルの基本をいくつか知っておくことが重要です。特に、コンピューターのディレクトリ構造を簡単にナビゲートするために、`cd`、`ls`、`pwd` を理解しておく必要があります。
* オペレーティング システムの [インストール手順](/docs/core/installation-overview) を使用して、dbt Core をインストールします。
* dbt Cloud シリーズのクイックスタートで、適切な設定とデータの読み込みの手順を完了します。たとえば、BigQuery の場合は、[設定 (BigQuery 内)](/guides/bigquery?step=2) と [データの読み込み (BigQuery)](/guides/bigquery?step=3) を完了します。
* まだお持ちでない場合は、[GitHub アカウントを作成](https://github.com/join) します。

### スタータープロジェクトを作成する

BigQuery を dbt で動作するように設定したら、独自のモデルを構築する前に、サンプル モデルを含むスターター プロジェクトを作成する準備が整います。

## リポジトリを作成する

次の手順では、このガイドの Git プロバイダーとして [GitHub](https://github.com/) を使用しますが、任意の Git プロバイダーを使用できます。[GitHub アカウントを作成](https://github.com/join) している必要があります。

1. [`dbt-tutorial` という名前の新しい GitHub リポジトリを作成](https://github.com/new) します。
2. リポジトリを他のユーザーと共有できるように、**パブリック** を選択します。後でいつでも非公開にすることができます。
3. その他のすべての設定はデフォルト値のままにします。
4. [リポジトリを作成] をクリックします。
5. [変更をコミット](https://docs.getdbt.com/guides/manual-install?step=6) で後で使用するために、「…or create a new repository on the command line」のコマンドを保存します。

## プロジェクトを作成する

ターミナルのコマンド ラインを使用して一連のコマンドを使用してプロジェクトを作成する方法を学びます。dbt Core には、dbt プロジェクトのスキャフォールディングに役立つ `init` コマンドが含まれています。

dbt プロジェクトを作成するには:

1. dbt Core がインストールされていることを確認し、`dbt --version` コマンドを使用してバージョンを確認します。

```shell
dbt --version
```

2. `init` コマンドを使用して `jaffle_shop` プロジェクトを開始します:

```shell
dbt init jaffle_shop
```

3. プロジェクトのディレクトリに移動します:

```shell
cd jaffle_shop
```

4. `pwd` を使用して、正しい場所にいることを確認します:

```shell
$ pwd
> Users/BBaggins/dbt-tutorial/jaffle_shop
```

5. Atom や VSCode などのコード エディターを使用して、前の手順で作成したプロジェクト ディレクトリ (jaffle_shop という名前) を開きます。このコンテンツには、フォルダーと、`init` コマンドによって生成された `.sql` ファイルおよび `.yml` ファイルが含まれます。

<div style={{maxWidth: '400px'}}>
<Lightbox src="/img/starter-project-dbt-cli.png" title="The starter project in a code editor" />
</div>

6. dbt は `dbt_project.yml` ファイルに次の値を提供します:

<File name='dbt_project.yml'>

```yaml
name: jaffle_shop # Change from the default, `my_new_project`

...

profile: jaffle_shop # Change from the default profile name, `default`

...

models:
    jaffle_shop: # Change from `my_new_project` to match the previous value for `name:`
    ...
```

</File>

## BigQuery に接続

ローカルで開発する場合、dbt は [profile](/docs/core/connect-data-platform/connection-profiles) を使用して <Term id="data-warehouse" /> に接続します。これは、ウェアハウスへの接続の詳細がすべて含まれた YAML ファイルです。

1. `~/.dbt/` ディレクトリに `profiles.yml` という名前のファイルを作成します。
2. BigQuery キーファイルをこのディレクトリに移動します。
3. 次の内容をコピーして、新しい profiles.yml ファイルに貼り付けます。記載されている値を更新してください。

<File name='profiles.yml'>

```yaml
jaffle_shop: # this needs to match the profile in your dbt_project.yml file
    target: dev
    outputs:
        dev:
            type: bigquery
            method: service-account
            keyfile: /Users/BBaggins/.dbt/dbt-tutorial-project-331118.json # replace this with the full path to your keyfile
            project: grand-highway-265418 # Replace this with your project id
            dataset: dbt_bbagins # Replace this with dbt_your_name, e.g. dbt_bilbo
            threads: 1
            timeout_seconds: 300
            location: US
            priority: interactive
```

</File>

4. プロジェクトから `debug` コマンドを実行して、正常に接続できることを確認します:

```shell
$ dbt debug
> Connection test: OK connection ok
```

<div style={{maxWidth: '400px'}}>
<Lightbox src="/img/successful-dbt-debug.png" title="A successful dbt debug command" />
</div>

### FAQs

<FAQ path="Warehouse/sample-profiles" alt_header="My data team uses a different data warehouse. What should my profiles.yml file look like for my warehouse?"/>
<FAQ path="Project/separate-profile" />
<FAQ path="Environments/profile-name" />
<FAQ path="Environments/target-names" />
<FAQ path="Environments/profile-env-vars" />

## 最初の dbt 実行を行います

サンプル プロジェクトにはいくつかのサンプル モデルが含まれています。これらを実行して、すべてが正常であることを確認します。

1. `run` コマンドを入力して、サンプル モデルを構築します:

```shell
dbt run
```

次のような出力が得られるはずです:

<div style={{maxWidth: '400px'}}>
<Lightbox src="/img/successful-dbt-run.png" title="A successful dbt run command" />
</div>

## 変更をコミットします

リポジトリに最新のコードが含まれるように変更をコミットします。

1. ターミナルで次のコマンドを実行して、作成した GitHub リポジトリを dbt プロジェクトにリンクします。リポジトリの正しい git URL を使用していることを確認してください。これは、[リポジトリの作成](https://docs.getdbt.com/guides/manual-install?step=2) の手順 5 で保存しておいたはずです。

```shell
git init
git branch -M main
git add .
git commit -m "Create a dbt project"
git remote add origin https://github.com/USERNAME/dbt-tutorial.git
git push -u origin main
```

2. GitHub リポジトリに戻り、新しいファイルが追加されたことを確認します。

### 最初のモデルを構築する

サンプル プロジェクトをセットアップしたら、楽しい部分である [モデルの構築](/docs/build/sql-models) に進むことができます。

次の手順では、サンプル クエリを取得して、それを dbt プロジェクト内のモデルに変換します。

## 新しい git ブランチをチェックアウトする

新しいコードで作業するために新しい git ブランチをチェックアウトします:

1. `checkout` コマンドを使用して `-b` フラグを渡し、新しいブランチを作成します:

```shell
$ git checkout -b add-customers-model
>  Switched to a new branch `add-customer-model`
```

## 最初のモデルを構築する


1. お気に入りのコード エディターでプロジェクトを開きます。
2. `models` ディレクトリに `models/customers.sql` という名前の新しい SQL ファイルを作成します。
3. 次のクエリを `models/customers.sql` ファイルに貼り付けます。

<Snippet path="tutorial-sql-query" />

4. コマンドラインから「dbt run」と入力します。
<div style={{maxWidth: '400px'}}>
<Lightbox src="/img/first-model-dbt-cli.png" title="A successful run with the dbt Core CLI" />
</div>

BigQuery コンソールに戻ると、このモデルから「select」できます。

### FAQs

<FAQ path="Runs/checking-logs" />
<FAQ path="Project/which-schema" />
<FAQ path="Models/create-a-schema" />
<FAQ path="Models/run-downtime" />
<FAQ path="Troubleshooting/sql-errors" />

## モデルの実現方法を変更する



<Snippet path="quickstarts/change-way-model-materialized" />

## サンプルモデルを削除する

<Snippet path="quickstarts/delete-example-models" />

## 他のモデルの上にモデルを構築する

<Snippet path="quickstarts/intro-build-models-atop-other-models" />

1. 元のクエリの `customers` CTE からの SQL を使用して、新しい SQL ファイル `models/stg_customers.sql` を作成します。
2. 元のクエリの `orders` CTE からの SQL を使用して、2 番目の新しい SQL ファイル `models/stg_orders.sql` を作成します。

<WHCode>

<div warehouse="BigQuery">

<File name='models/stg_customers.sql'>

```sql
select
    id as customer_id,
    first_name,
    last_name

from `dbt-tutorial`.jaffle_shop.customers
```

</File>

<File name='models/stg_orders.sql'>

```sql
select
    id as order_id,
    user_id as customer_id,
    order_date,
    status

from `dbt-tutorial`.jaffle_shop.orders
```

</File>

</div>

<div warehouse="Databricks">

<File name='models/stg_customers.sql'>

```sql
select
    id as customer_id,
    first_name,
    last_name

from jaffle_shop_customers
```

</File>

<File name='models/stg_orders.sql'>

```sql
select
    id as order_id,
    user_id as customer_id,
    order_date,
    status

from jaffle_shop_orders
```

</File>

</div>

<div warehouse="Redshift">

<File name='models/stg_customers.sql'>

```sql
select
    id as customer_id,
    first_name,
    last_name

from jaffle_shop.customers
```

</File>

<File name='models/stg_orders.sql'>

```sql
select
    id as order_id,
    user_id as customer_id,
    order_date,
    status

from jaffle_shop.orders
```

</File>

</div>

<div warehouse="Snowflake">

<File name='models/stg_customers.sql'>

```sql
select
    id as customer_id,
    first_name,
    last_name

from raw.jaffle_shop.customers
```

</File>

<File name='models/stg_orders.sql'>

```sql
select
    id as order_id,
    user_id as customer_id,
    order_date,
    status

from raw.jaffle_shop.orders
```

</File>

</div>

</WHCode>

3. `models/customers.sql` ファイル内の SQL を次のように編集します:

<File name='models/customers.sql'>

```sql
with customers as (

    select * from {{ ref('stg_customers') }}

),

orders as (

    select * from {{ ref('stg_orders') }}

),

customer_orders as (

    select
        customer_id,

        min(order_date) as first_order_date,
        max(order_date) as most_recent_order_date,
        count(order_id) as number_of_orders

    from orders

    group by 1

),

final as (

    select
        customers.customer_id,
        customers.first_name,
        customers.last_name,
        customer_orders.first_order_date,
        customer_orders.most_recent_order_date,
        coalesce(customer_orders.number_of_orders, 0) as number_of_orders

    from customers

    left join customer_orders using (customer_id)

)

select * from final

```

</File>

4. `dbt run` を実行します。

今回、`dbt run` を実行すると、`stg_customers`、`stg_orders`、`customers` の個別のビュー/テーブルが作成されました。dbt はこれらのモデルを実行する順序を推測しました。`customers` は `stg_customers` と `stg_orders` に依存しているため、dbt は `customers` を最後に構築します。これらの依存関係を明示的に定義する必要はありません。

### FAQs {#faq-2}

<FAQ path="Runs/run-one-model" />
<FAQ path="Project/unique-resource-names" />
<FAQ path="Project/structure-a-project" alt_header="As I create more models, how should I keep my project organized? What should I name my models?" />

### 次のステップ

<Snippet path="tutorial-next-steps-1st-model" />

以下も参照できます:

* `target` ディレクトリでは、コンパイルされたすべての SQL を確認できます。`run` ディレクトリでは、実行中のテーブル作成または置換ステートメントが表示されます。これは、正しい DDL でラップされた選択ステートメントです。
* `logs` ファイルでは、dbt Core がプロジェクト内で発生するすべてのアクションをどのようにログに記録するかを確認できます。実行中の選択ステートメントと、dbt の実行時に発生する Python ログが表示されます。

## モデルにテストを追加する

<Snippet path="tutorial-add-tests-to-models" />

## モデルを文書化する

<Snippet path="tutorial-document-your-models" />

3. `dbt docs serve` コマンドを実行して、ローカル Web サイトでドキュメントを起動します。

#### FAQs

<FAQ path="Docs/long-descriptions" />
<FAQ path="Docs/sharing-documentation" />


#### Next steps

<Snippet path="tutorial-next-steps-tests" />

## 更新された変更をコミットする

リポジトリに最新のコードが含まれるように、プロジェクトに加えた変更をコミットする必要があります。

1. すべての変更を git に追加します: `git add -A`
2. 変更をコミットします: `git commit -m "顧客モデル、テスト、ドキュメントの追加"`
3. 変更をリポジトリにプッシュします: `git push`
4. リポジトリに移動し、プル リクエストを開いてコードをマスター ブランチにマージします。

## ジョブをスケジュールする

[ジョブをデプロイ](/docs/deploy/deployments)し、本番環境で dbt プロジェクトを自動化する最も簡単で信頼性の高い方法として、dbt Cloud を使用することをお勧めします。

開始方法の詳細については、[ジョブの作成とスケジュール](/docs/deploy/deploy-jobs#create-and-schedule-jobs) を参照してください。

<Lightbox src="/img/docs/dbt-cloud/deployment/run-overview.jpg" width="90%" title="Overview of a dbt Cloud job run, which includes the job run details, trigger type, commit SHA, environment name, detailed run steps, logs, and more."/>

dbt Core を使用してジョブをスケジュールする方法の詳細については、[dbt airflow](/blog/dbt-airflow-spiritual-alignment) のブログ投稿を参照してください。

</div>
