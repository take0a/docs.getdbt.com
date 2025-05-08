---
title: "dbt clone コマンドについて"
sidebar_label: "clone"
id: "clone"
---

`dbt clone` コマンドは、選択したノードを [指定された状態](/reference/node-selection/syntax#establishing-state) からターゲットスキーマに複製します。このコマンドは、`clone` マテリアライゼーションを使用します。
- データプラットフォームがテーブルのゼロコピー複製をサポートしており (Snowflake、Databricks、または BigQuery)、このモデルがソース環境にテーブルとして存在する場合、dbt はそれをターゲット環境にクローンとして作成します。
- それ以外の場合、dbt は単純なポインタビュー (ソースオブジェクトから `select *`) を作成します。
- デフォルトでは、`dbt clone` は現在のターゲットに既存のリレーションを再作成しません。これを無効にするには、`--full-refresh` フラグを使用します。
- 個々のクローンステートメントは互いに独立しているため、実行時間を短縮するために [スレッド](/docs/running-a-dbt-project/using-threads) の数を増やすことを推奨します。

`clone` コマンドは、次のような場合に役立ちます。
- Blue/Green 継続的デプロイメント（ゼロコピー クローニング テーブルをサポートするデータ ウェアハウスの場合）
- 現在の本番環境状態を開発スキーマにクローニングする
- dbt Cloud CI ジョブで増分モデルを処理する（ゼロコピー クローニング テーブルをサポートするデータ ウェアハウスの場合）
- BI ツールで下流の依存関係におけるコード変更をテストする


```bash
# clone all of my models from specified state to my target schema(s)
dbt clone --state path/to/artifacts

# clone one_specific_model of my models from specified state to my target schema(s)
dbt clone --select "one_specific_model" --state path/to/artifacts

# clone all of my models from specified state to my target schema(s) and recreate all pre-existing relations in the current target
dbt clone --state path/to/artifacts --full-refresh

# clone all of my models from specified state to my target schema(s), running up to 50 clone statements in parallel
dbt clone --state path/to/artifacts --threads 50
```

### [defer](/reference/node-selection/defer)ではなく `dbt clone` を使用するべき状況とは？

defer とは異なり、`dbt clone` ではある程度の計算処理とデータウェアハウスへの追加オブジェクトの作成が必要です。多くの場合、defer は `dbt clone` よりも安価でシンプルな代替手段となります。ただし、`dbt clone` はdefer が不可能なユースケースにも対応します。

例えば、`dbt clone` は実際のデータウェアハウスオブジェクトを作成することで、dbt 外部の下流依存関係（BI ツールなど）でコード変更をテストできます。

別の例として、ゼロコピークローンをサポートするウェアハウスでコストのかかる `full-refresh` ビルドを回避するために、dbt Cloud CI ジョブの最初のステップとして、変更した増分モデルを `clone` することができます。

## dbt Cloud でのクローン作成

dbt Cloud では、`dbt clone` コマンドを使用して、状態間でノードをクローンできます。このコマンドは [dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) と [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) で利用でき、[`--defer`](/reference/node-selection/defer) 機能を使用します。dbt Cloud での defer の詳細については、[dbt Cloud での defer の使用](/docs/cloud/about-cloud-develop-defer) をご覧ください。

- **dbt Cloud CLI の使用** &mdash; dbt Cloud CLI の `dbt clone` コマンドには、`--defer` フラグが自動的に含まれます。つまり、追加の設定なしで `dbt clone` コマンドを使用できます。

- **dbt Cloud IDE の使用** &mdash; dbt Cloud IDE で `dbt clone` コマンドを使用するには、`dbt clone` コマンドを実行する前に以下の手順を実行してください。

  - **本番環境** をセットアップし、ジョブを正常に実行します。
  - コマンドバーの右下にあるスイッチを切り替えて、**本番環境への延期** を有効にします。
  <Lightbox src="/img/docs/dbt-cloud/defer-toggle.jpg" width="80%" title="dbt Cloud IDE で延期を有効にするには、コマンドバーの右下にある [本番環境への延期] トグルを選択します。"/>
  - コマンドバーから `dbt clone` コマンドを実行します。

`dbt clone` と defer のどちらを使用するかについてのベストプラクティスの詳細については、[こちらの開発者ブログ投稿](https://docs.getdbt.com/blog/to-defer-or-to-clone) をご覧ください。
