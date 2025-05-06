---
title: "一般的なクエリをキャッシュする"
id: "sl-cache"
description: "一般的なクエリをキャッシュしてパフォーマンスを高速化し、クエリの計算を削減します。"
tags: [Semantic Layer]
sidebar_label: "Cache common queries"
---


dbt セマンティックレイヤーを使用すると、よく使用するクエリをキャッシュしてパフォーマンスを向上させ、高負荷なクエリの計算負荷を軽減できます。

キャッシュには2つの種類があります。

- [結果キャッシュ](#result-caching) は、データプラットフォームに組み込まれているキャッシュレイヤーを活用します。
- [宣言型キャッシュ](#declarative-caching) は、保存されたクエリ設定を使用してキャッシュを事前にウォームアップできます。

キャッシュを使用するとクエリを高速化し、計算時間を短縮できますが、この2つの違いはユースケースによって異なります。

- 結果のキャッシュは、データプラットフォームのキャッシュを活用して自動的に行われます。
- 宣言型キャッシュでは、キャッシュするクエリを具体的に「宣言」できます。宣言型キャッシュを使用する場合は、どのクエリをキャッシュするかを事前に予測する必要があります。
- 宣言型キャッシュでは、キャッシュによるパフォーマンス上のメリットを損なうことなく、ダッシュボードを動的にフィルタリングすることもできます。これは、ディメンションのフィルタ（保存済みクエリ設定に既に存在するもの）がキャッシュを使用するためです。

## 前提条件
- dbt Cloud [Team または Enterprise](https://www.getdbt.com/) プラン。
- dbt Cloud 環境は、従来の dbt Core バージョンではなく、[リリーストラック](/docs/dbt-versions/cloud-release-tracks) である必要があります。
- ジョブが正常に実行され、[本番環境](/docs/deploy/deploy-environments#set-as-production-environment) が設定されている必要があります。
- 宣言型キャッシュを使用するには、[保存済みクエリ](/docs/build/saved-queries) YAML 構成ファイルに [エクスポート](/docs/use-dbt-semantic-layer/exports) が定義されている必要があります。

## 結果のキャッシュ

結果のキャッシュは、データプラットフォームに組み込まれているキャッシュレイヤーと機能を活用します。[MetricFlow](/docs/build/about-metricflow) は複数のクエリリクエストに対して同じ SQL を生成するため、データプラットフォームのキャッシュを活用できます。データプラットフォームの仕様をよくご確認ください。

Snowflake を例に、キャッシュの仕組みを説明します。他のデータプラットフォームでも同様の仕組みです。

1. **コールドキャッシュから実行** - BI ツールから、過去 24 時間以内に実行されていないセマンティックレイヤークエリを実行すると、クエリはデータセット全体をスキャンし、キャッシュは使用しません。
2. **ウォームキャッシュから実行** - 1 時間後に同じクエリを再実行した場合、Snowflake で生成および実行された SQL は同じままです。Snowflake では、結果キャッシュはユーザーごとに 24 時間設定されるため、繰り返し実行されるクエリでキャッシュが使用され、結果がより速く返されます。

データプラットフォームによって、キャッシュレイヤーやキャッシュ無効化ルールが異なる場合があります。一般的なデータプラットフォームにおけるキャッシュの仕組みに関する参考資料を以下に示します:

- [BigQuery](https://cloud.google.com/bigquery/docs/cached-results)
- [DataBricks](https://docs.databricks.com/en/optimizations/disk-cache.html)
- [Microsoft Fabric](https://learn.microsoft.com/en-us/fabric/data-warehouse/caching)
- [Redshift](https://docs.aws.amazon.com/redshift/latest/dg/c_challenges_achieving_high_performance_queries.html#result-caching)
- [Snowflake](https://community.snowflake.com/s/article/Caching-in-the-Snowflake-Cloud-Data-Platform)
- [Starburst Galaxy](https://docs.starburst.io/starburst-galaxy/data-engineering/optimization-performance-and-quality/workload-optimization/warp-speed-enabled.html)

## 宣言型キャッシュ

宣言型キャッシュを使用すると、[保存済みクエリ](/docs/build/saved-queries)を使用してキャッシュを事前ウォームアップできます。そのためには、`saved_queries` 設定でキャッシュ構成を `true` に設定します。これは、主要なダッシュボードや一般的なアドホッククエリリクエストのパフォーマンスを最適化するのに役立ちます。

:::tip
宣言型キャッシュを使用すると、キャッシュによるパフォーマンス上のメリットを損なうことなく、ダッシュボードを動的にフィルタリングできます。これは、ディメンションのフィルタ（保存済みクエリ設定に既に存在するもの）がキャッシュを使用するためです。

例えば、ダッシュボードで地理的な地域でメトリックをフィルタリングする場合、クエリはキャッシュにアクセスし、より高速な結果を得ることができます。また、静的フィルタを使用した保存済みクエリを別途作成する必要もありません。
:::

設定の詳細については、[宣言型キャッシュの設定](#declarative-caching-setup)を参照してください。

宣言型キャッシュの仕組み:
- 保存済みクエリのYAML設定ファイルに[エクスポート](/docs/use-dbt-semantic-layer/exports)が定義されていることを確認してください。
- 保存済みクエリを実行すると、dbtセマンティックレイヤーがトリガーされ、次の処理が実行されます。
  - エクスポートが定義された保存済みクエリから、データプラットフォームにキャッシュされたテーブルを構築します。
  - 保存済みクエリの入力に一致するクエリリクエストでキャッシュが使用されるようにし、より迅速にデータを返します。
  - キャッシュされたテーブル内の指標に関連する上流モデルで新しい最新データが検出されると、キャッシュが自動的に無効化されます。
  - 保存済みクエリを次回実行するときに、キャッシュが更新（または再構築）されます。
 
<details>

<summary> 📹 宣言型キャッシュがどのように機能するかを確認するには、このビデオ デモをご覧ください。</summary>

このビデオでは、宣言型キャッシュの概念、dbt Cloud スケジューラを使用してそれを実行する方法、そしてその結果としてダッシュボードがどれだけ速く読み込まれるかについて説明します。

<LoomVideo id='aea82a4dee364dfdb536e7b8068684e7' />

</details>

dbt セマンティック レイヤーがクエリ要求を受信したときに何が起こるかを示す次の図を参照してください:

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/declarative-cache-query-flow.jpg" width="70%" title="Overview of the declarative cache query flow" />

### 宣言型キャッシュの設定

キャッシュにデータを取り込むには、保存済みクエリのYAMLファイル設定でエクスポートを設定し、`cache config` を `true` に設定する必要があります。エクスポートが定義されていない保存済みクエリをキャッシュすることはできません。

<File name='semantic_model.yml'>

```yaml
saved_queries:
  - name: my_saved_query
    ... # Rest of the saved queries configuration.
    config:
      cache:
        enabled: true  # Set to true to enable, defaults to false.
    exports:
      - name: order_data_key_metrics
        config:
          export_as: table
```
</File>

プロジェクトレベルで保存クエリを有効にするには、[`dbt_project.yml` ファイル](/reference/dbt_project.yml) で `saved-queries` 設定を設定します。これにより、各ファイルで保存クエリを設定する手間が省けます:

<File name='dbt_project.yml'>

```yaml
saved-queries:
  my_saved_query:
    config:
      +cache:
        enabled: true
```
</File>

### 宣言型キャッシュを実行する

YAML 構成で宣言型キャッシュを設定したら、dbt Cloud ジョブスケジューラで [exports](/docs/use-dbt-semantic-layer/exports) を実行し、保存済みクエリからデータプラットフォームにキャッシュされたテーブルを構築できます。

- [exports を使用してジョブを設定する](/docs/use-dbt-semantic-layer/exports) を使用して、保存済みクエリを dbt Cloud で実行します。
- dbt Semantic Layer は、専用の `dbt_sl_cache` スキーマを使用して、データプラットフォームにキャッシュテーブルを構築します。
- キャッシュスキーマとテーブルは、デプロイメント認証情報を使用して作成されます。Semantic Layer ユーザーにこのスキーマへの読み取りアクセス権を付与する必要があります。
- キャッシュは、保存済みクエリジョブと同じスケジュールで更新（または再構築）されます。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/cache-creation-flow.jpg" width="70%" title="Overview of the cache creation flow." />

ジョブの実行が成功したら、ダッシュボードに戻って宣言型キャッシュの速度と利点を体験できます。

## キャッシュ管理

dbt Cloud は、dbt モデル実行から得られたメタデータを使用して、キャッシュの無効化をインテリジェントに管理します。dbt ジョブを開始すると、前回のモデル実行時間が記録され、キャッシュの上流にあるメトリクスの最新状態がチェックされます。

上流モデルに、キャッシュ作成後に作成されたデータが含まれている場合、dbt Cloud はキャッシュを無効化します。つまり、クエリは古いケースを使用せず、ソースデータから直接クエリを実行します。古くなったキャッシュテーブルは定期的に削除され、保存されたクエリが次に実行されるときに、dbt Cloud は新しいキャッシュを書き込みます。

[dbt セマンティック レイヤー API](/docs/dbt-cloud-apis/sl-api-overview) の `InvalidateCacheResult` フィールドを使用して、キャッシュを手動で無効化できます。

## FAQs

<DetailsToggle alt_header="キャッシュはアクセス制御とどのように相互作用しますか?">

キャッシュされたデータは、基盤となるモデルとは別に保存されます。メトリクスがキャッシュから取得された場合、クエリ実行時にそれらのテーブルにセキュリティコンテキストが適用されません。

今後、認証情報を複製し、必要な最小限のアクセスレベルを特定し、それらの権限をキャッシュされたテーブルに適用する予定です。

</DetailsToggle>


## 関連ドキュメント
- [CI でセマンティックノードを検証する](/docs/deploy/ci-jobs#semantic-validations-in-ci)
- [保存されたクエリ](/docs/build/saved-queries)
- [dbt セマンティックレイヤーに関するよくある質問](/docs/use-dbt-semantic-layer/sl-faqs)
