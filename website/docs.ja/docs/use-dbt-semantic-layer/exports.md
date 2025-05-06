---
title: "エクスポートを使用してクエリを作成する"
description: "エクスポートを使用して、スケジュールに従ってテーブルをデータ プラットフォームに書き込みます。"
sidebar_label: "Write queries with exports"
keywords: [DBT_INCLUDE_SAVED_QUERY, exports, DBT_EXPORTS_SAVED_QUERY, dbt Cloud, Semantic Layer]
---

エクスポートは、[保存済みクエリ](/docs/build/saved-queries) の機能を強化します。保存済みクエリを実行し、その出力をデータ プラットフォーム内のテーブルまたはビューに書き込むことで、この機能を強化します。保存済みクエリは、MetricFlow でよく使用されるクエリを保存して再利用する方法ですが、エクスポートではこの機能をさらに強化し、次のことが可能になります。

- dbt Cloud ジョブ スケジューラを使用して、データ プラットフォーム内でこれらのクエリを記述できるようになります。
- 指標とディメンションのテーブルを公開することで、dbt セマンティック レイヤーをネイティブにサポートしていないツールとの統合パスを提供します。

基本的に、エクスポートはデータ プラットフォーム内の他のテーブルと同様です。つまり、エクスポートを使用することで、任意の SQL インターフェースを介して指標定義をクエリしたり、高度な [セマンティック レイヤー統合](/docs/cloud-integrations/avail-sl-integrations) を使用せずに下流のツールに接続したりできます。エクスポートを実行すると、[クエリされた指標](/docs/cloud/billing#what-c​​ounts-as-a-queried-metric) の使用量としてカウントされます。エクスポートからの結果のテーブルまたはビューをクエリしても、クエリされたメトリックの使用にはカウントされません。

## 前提条件

- [Team または Enterprise](https://www.getdbt.com/pricing/) プランの dbt Cloud アカウントをお持ちであること。
- Snowflake、BigQuery、Databricks、Redshift、Postgres のいずれかのデータプラットフォームを使用していること。
- [dbt バージョン](/docs/dbt-versions/upgrade-dbt-version-in-cloud) が 1.7 以降であること。
- dbt プロジェクトで dbt セマンティック レイヤーが [構成済み](/docs/use-dbt-semantic-layer/setup-sl) であること。
- [ジョブ スケジューラ](/docs/deploy/job-scheduler) が有効になっている dbt Cloud 環境があること。
- dbt プロジェクトに [保存済みクエリ](/docs/build/saved-queries) と [エクスポート設定済み](/docs/build/saved-queries#configure-exports) があります。設定で [キャッシュ](/docs/use-dbt-semantic-layer/sl-cache) を活用して、よく使用するクエリをキャッシュし、パフォーマンスを向上させ、コンピューティングコストを削減してください。
- [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) がインストールされています。なお、エクスポートは dbt Cloud IDE ではまだサポートされていません。

## エクスポートのメリット

次のセクションでは、エクスポートを使用する主なメリットについて説明します:

<Expandable alt_header="DRY表現">

現在、テーブルの作成には、数十、数百、あるいは数千ものテーブルを作成し、データをサマリーテーブルやメトリックマートテーブルに非正規化する作業が必要になることがよくあります。エクスポートの主なメリットは、各メトリック、ディメンション、結合、フィルターなどを構築するロジックを「Don't Repeat Yourself（DRY）」形式で表現できることです。これにより、手動で記述したSQLモデルを保存済みクエリ内のメトリックやディメンションへの参照に置き換える場合でも、これらのコンポーネントを再利用して長期的なスケーラビリティを実現できます。
</Expandable>

<Expandable alt_header="より簡単な変更">

エクスポートにより、指標とディメンションへの変更は一箇所で行われ、その後、様々な出力先にシームレスに反映されます。これにより、同じ概念を参照するすべてのモデルで指標を更新する必要がなくなります。
</Expandable>

<Expandable alt_header="キャッシング"> 

エクスポートを使用してキャッシュを事前に入力し、動的なセマンティック レイヤー API を通じてユーザーに提供するために必要なものを事前に計算できるようにします。
</Expandable>

#### 考慮事項

エクスポートには多くのメリットがありますが、メリットに含まれないユースケースについても注意が必要です。
- ビジネスユーザーは、数十、数百、あるいは数千ものテーブルからデータを取得するのに苦労する可能性があり、適切なテーブルを選択するのが難しい場合があります。
- ビジネスユーザーは、事前に構築されたテーブルからデータを集計およびフィルタリングする際に、ミスを犯す可能性があります。

これらのユースケースでは、エクスポートではなく、動的な[dbtセマンティックレイヤーAPI](/docs/dbt-cloud-apis/sl-api-overview)を使用してください。

## エクスポートを実行する

開発環境または本番環境でエクスポートを実行する前に、dbt プロジェクトで [保存済みクエリとエクスポートを設定](/docs/build/saved-queries) する必要があります。保存済みクエリ設定では、dbt Cloud ジョブスケジューラによる [キャッシュ](/docs/use-dbt-semantic-layer/sl-cache) を活用して、よく使用するクエリをキャッシュし、パフォーマンスを向上させ、コンピューティングコストを削減することもできます。

エクスポートを実行する方法は 2 つあります。

- [開発環境でエクスポートを実行](#exports-in-development) [dbt Cloud CLI](/docs/cloud/cloud-cli-installation) を使用して、本番環境へのデプロイ前に出力をテストします (dbt Cloud IDE でエクスポートを設定できますが、IDE で直接実行することはまだサポートされていません)。dbt Cloud IDE を使用している場合は、`dbt build` を使用してエクスポートを実行します。 [環境変数](#set-environment-variable)が有効になっていることを確認してください。
- [dbt Cloud ジョブ スケジューラ](/docs/deploy/job-scheduler)を使用して[本番環境でエクスポートを実行](#exports-in-production)し、データ プラットフォーム内でこれらのクエリを記述します。

## 開発環境でのエクスポート

本番環境に移行する前にエクスポートの出力をテストしたい場合は、開発認証情報を使用して開発環境でエクスポートを実行できます。

このセクションでは、開発環境でエクスポートを実行するために使用できるさまざまなコマンドとオプションについて説明します。

- 単一の保存済みクエリに対して開発環境でエクスポートをテストおよび生成するには、[`dbt sl export` コマンド](#exports-for-single-saved-query) を使用します。また、`--select` フラグを使用して、保存済みクエリから特定のエクスポートを指定することもできます。

- 複数の保存済みクエリに対してエクスポートを一度に実行するには、[`dbt sl export-all` コマンド](#exports-for-multiple-saved-queries) を使用します。このコマンドを使用すると、複数のクエリのエクスポートを同時に管理および実行できるため、時間と労力を節約できます。

### 保存済みの単一クエリのエクスポート

dbt Cloud CLI でエクスポートを実行するには、次のコマンドを使用します:

```bash
dbt sl export
```

次の表は、`--` フラグ プレフィックスを使用してパラメータを指定する `dbt sl export` コマンドのオプションを示しています:

| Parameters | Type    | Required | Description    |
| ------- | --------- | ---------- | ---------------- |
| `name` | String    | Required     |`export` オブジェクトの名前。 |
| `saved-query` | String    | Required     | 使用できる保存済みクエリの名前。 |
| `select` | List or String   | Optional    | 保存されたクエリから選択するエクスポートの名前を指定します。 |
| `exclude` | String  | Optional    | 保存されたクエリから除外するエクスポートの名前を指定します。 |
| `export_as` | String  | Optional    | 設定で使用可能な `export_as` タイプから作成するエクスポートのタイプ。使用可能なオプションは `table` または `view` です。 |
| `schema` | String  | Optional    | テーブルまたはビューの作成に使用するスキーマ。 |
| `alias` | String  | Optional    | テーブルまたはビューの書き込みに使用するテーブル別名。 |

保存したクエリに定義されているエクスポートを実行し、開発環境でテーブルまたはビューを書き込むこともできます。以下のコマンド例と出力を参照してください:

```bash
dbt sl export --saved-query sq_name
```

出力は次のようになります:

```bash
Polling for export status - query_id: 2c1W6M6qGklo1LR4QqzsH7ASGFs..
Export completed.
```

### select フラグを使用する

保存済みクエリには複数のエクスポートを指定でき、デフォルトではすべてのエクスポートが実行されます。[開発](#exports-in-development) の `select` フラグを使用すると、特定のエクスポートまたは複数のエクスポートを選択できます。保存済みクエリからメトリックやディメンションをサブ選択することはできません。これは、エクスポート設定（テーブル形式やスキーマなど）を変更するためだけに使用できます。

例えば、次のコマンドは `export_1` と `export_2` を実行しますが、`--alias` フラグや `--export_as` フラグは指定できません。

```bash
dbt sl export --saved-query sq_name --select export_1,export2
```

<details>
<summary>エクスポート設定の上書き</summary>

`--select` フラグは主に、特定のエクスポートを含めるか除外するかを指定します。これらの設定を変更する必要がある場合は、以下のフラグを使用してエクスポート設定をオーバーライドできます。

- `--export-as` - エクスポートのマテリアライズタイプ（テーブルまたはビュー）を定義します。これにより、独自の設定を持つ新しいエクスポートが作成され、開発中のテストに役立ちます。
- `--schema` - 書き込まれるテーブルまたはビューに使用するスキーマを指定します。
- `--alias` - 書き込まれるテーブルまたはビューにカスタムエイリアスを割り当てます。これにより、デフォルトのエクスポート名がオーバーライドされます。

注意：`--select` フラグは `alias` または `schema` と一緒に使用できません。

例えば、次のコマンドを使用して、`new_export` という名前の新しいエクスポートをテーブルとして作成できます。

```bash
dbt sl export --saved-query sq_number1 --export-as table --alias new_export
```
</details>

### 複数の保存済みクエリのエクスポート

`dbt sl export-all` コマンドを使用すると、複数の保存済みクエリのエクスポートを一度に実行できます。これは、単一の保存済みクエリのエクスポートのみを実行する `dbt sl export` コマンドとは異なります。たとえば、複数の保存済みクエリのエクスポートを実行するには、次のようにします:

```bash
dbt sl export-all
```

出力は次のようになります:

```bash
Exports completed:
- Created TABLE at `DBT_SL_TEST.new_customer_orders`
- Created VIEW at `DBT_SL_TEST.new_customer_orders_export_alias`
- Created TABLE at `DBT_SL_TEST.order_data_key_metrics`
- Created TABLE at `DBT_SL_TEST.weekly_revenue`

Polling completed
```

コマンド `dbt sl export-all` は、単一のコマンドで複数のエクスポートを管理する柔軟性を提供します。

## 本番環境でのエクスポート

dbt Cloud でエクスポートを有効にして実行すると、データワークフローが最適化され、リアルタイムのデータアクセスが確保されます。これにより、効率性とガバナンスが向上し、よりスマートな意思決定が可能になります。

エクスポートでは、本番環境のデフォルトの認証情報が使用されます。エクスポートを有効にして保存済みのクエリを実行し、データプラットフォーム内で書き込むには、次の手順を実行します。

1. dbt Cloud で [環境変数を設定](#set-environment-variable)します。
2. [エクスポートジョブを作成して実行](#create-and-execute-exports)します。

### 環境変数を設定する
<!-- for version 1.7 -->
<VersionBlock firstVersion lastVersion="1.7">

1. Click **Deploy** in the top navigation bar and choose **Environments**.
2. Select **Environment variables**.
3. [Set the environment variable](/docs/build/environment-variables#setting-and-overriding-environment-variables) key to `DBT_INCLUDE_SAVED_QUERY` and the environment variable's value to `TRUE` (`DBT_INCLUDE_SAVED_QUERY=TRUE`).

This ensures saved queries and exports are included in your dbt build job. For example, running `dbt build --select sq_name` runs the equivalent of `dbt sl export --saved-query sq_name` in the dbt Cloud Job scheduler. 

If exports aren't needed, you can set the value(s) to `FALSE` (`DBT_INCLUDE_SAVED_QUERY=FALSE`).

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/deploy_exports.jpg" width="90%" title="Add an environment variable to run exports in your production run." />

</VersionBlock>

<!-- for Release Tracks -->
<VersionBlock firstVersion="1.8">

1. 上部のナビゲーションバーで  **Deploy** をクリックし、**Environments** を選択します。
2. **Environment variables** を選択します。
3. [環境変数](/docs/build/environment-variables#setting-and-overriding-environment-variables) キーを `DBT_EXPORT_SAVED_QUERIES` に設定し、環境変数の値を `TRUE` (`DBT_EXPORT_SAVED_QUERIES=TRUE`) に設定します。
*注: dbt v1.7 を使用している場合は、環境変数キーを `DBT_INCLUDE_SAVED_QUERY` に設定してください。詳細を表示するには、ドキュメント切り替えを使用してバージョン「1.7」を選択してください。

これにより、保存されたクエリとエクスポートが dbt ビルドジョブに含まれるようになります。たとえば、`dbt build -s sq_name` を実行すると、dbt Cloud Job Scheduler で `dbt sl export --saved-query sq_name` と同等の機能が実行されます。

エクスポートが不要な場合は、値を `FALSE` に設定できます（`DBT_EXPORT_SAVED_QUERIES=FALSE`）。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/env-var-dbt-exports.jpg" width="90%" title="Add an environment variable to run exports in your production run." />

</VersionBlock>

ビルドジョブを実行すると、そのジョブ内の dbt モデルの下流にある保存済みのクエリも実行されます。エクスポートデータが最新であることを確認するには、下流ステップ（モデルの後に）としてエクスポートを実行してください。

### エクスポートの作成と実行
<VersionBlock firstVersion lastVersion="1.7">

1. Create a [deploy job](/docs/deploy/deploy-jobs) and ensure the `DBT_INCLUDE_SAVED_QUERY=TRUE` environment variable is set, as described in [Set environment variable](#set-environment-variable).
   - This enables you to run any export that needs to be refreshed after a model is built.
   - Use the [selector syntax](/reference/node-selection/syntax) `--select` or `-s` option in your build command to specify a particular dbt model or saved query to run. For example, to run all saved queries downstream of the `orders` semantic model, use the following command:
    ```bash
      dbt build --select orders+
      ```

</VersionBlock>

<VersionBlock firstVersion="1.8">

1. [デプロイジョブ](/docs/deploy/deploy-jobs)を作成し、[環境変数の設定](#set-environment-variable)の説明に従って、`DBT_EXPORT_SAVED_QUERIES=TRUE`環境変数が設定されていることを確認します。
  - これにより、モデルの構築後に更新が必要なエクスポートを実行できるようになります。
  - ビルドコマンドで[セレクタ構文](/reference/node-selection/syntax)の`--select`または`-s`オプションを使用して、実行する特定のdbtモデルまたは保存済みクエリを指定します。たとえば、`orders`セマンティックモデルの下流にあるすべての保存済みクエリを実行するには、次のコマンドを使用します:
    ```bash
      dbt build --select orders+
      ```

</VersionBlock>

2. dbt がモデルの構築を完了すると、MetricFlow サーバーはエクスポートを処理し、必要な SQL をコンパイルして、データプラットフォームに対してこの SQL を実行します。データはデータプラットフォーム内に保持されるため、「create table」ステートメントが直接実行されます。
3. ジョブログでエクスポートの実行詳細を確認し、エクスポートが正常に実行されたことを確認します。これはトラブルシューティングと精度の確保に役立ちます。保存されたクエリは dbt DAG に統合されているため、エクスポートに関連するすべての出力はジョブログで確認できます。
4. これで、データプラットフォームでクエリを実行できるようになりました。🎉

## FAQs

<DetailsToggle alt_header="1 つの保存されたクエリで複数のエクスポートを実行できますか?">

はい、可能です。ただし、エクスポートの名前、スキーマ、およびマテリアライズ戦略が異なります。

</DetailsToggle>

<DetailsToggle alt_header="保存したクエリのすべてのエクスポートを実行するにはどうすればよいですか?">

- 本番環境では、ビルド コマンドで直接呼び出して保存済みクエリをビルドするか、モデルとそのモデルの下流にあるエクスポートをビルドできます。
- 開発環境では、`dbt sl export --saved-query sq_name` を実行してすべてのエクスポートを実行できます。

</DetailsToggle>

<DetailsToggle alt_header="保存したクエリの下流に複数のモデルがある場合、重複したエクスポートが実行されますか?">

dbt は、保存済みクエリの下流に複数のモデルを構築する場合でも、各エクスポートを 1 回だけ実行します。例えば、「order_metrics」という保存済みクエリがあり、このクエリには「orders」と「order_items」セマンティックモデルの両方のメトリクスが含まれているとします。

「dbt build」を使用すれば、両方のモデルを含むジョブを実行できます。この場合、「orders」と「order_items」の両方のモデルが実行されますが、「order_metrics」エクスポートは 1 回だけ実行されます。
</DetailsToggle>

<DetailsToggle alt_header="ref() を使用してエクスポートを dbt モデルとして参照できますか?">

いいえ、`ref` を使用してエクスポートを参照することはできません。エクスポートは DAG のリーフノードとして扱われます。エクスポートを変更すると、セマンティックレイヤーの元のメトリクスとの不整合が発生する可能性があります。

</DetailsToggle>

<DetailsToggle alt_header="エクスポートは、PowerBI などの dbt セマンティック レイヤーをサポートしていないツールで dbt セマンティック レイヤーを使用するのにどのように役立ちますか?">

エクスポートは、データプラットフォーム内のメトリックとディメンションのテーブルを公開することで、dbt セマンティックレイヤーにネイティブに接続できないツールに統合パスを提供します。

エクスポートを使用することで、PowerBI などのツールとのカスタム統合を作成できます。

</DetailsToggle>

<DetailsToggle alt_header="リソースタイプ別に saved_queries を選択するにはどうすればよいですか?">

保存されているすべてのクエリを dbt ビルド実行に含めるには、[`--resource-type` フラグ](/reference/global-configs/resource-type) を使用して、コマンド `dbt build --resource-type saved_query` を実行します。

</DetailsToggle>

## 関連ドキュメント
- [CI ジョブでセマンティックノードを検証する](/docs/deploy/ci-jobs#semantic-validations-in-ci)
- [キャッシュ](/docs/use-dbt-semantic-layer/sl-cache)の設定
- [dbt セマンティックレイヤーに関するよくある質問](/docs/use-dbt-semantic-layer/sl-faqs)