---
title: "Query the Discovery API"
id: "discovery-querying"
sidebar_label: "Query the Discovery API"
pagination_next: "docs/dbt-cloud-apis/discovery-schema-environment"
---

Discovery API はアドホッククエリと統合をサポートしています。API を初めてご利用になる場合は、[Discovery API について](/docs/dbt-cloud-apis/discovery-api) の概要をご覧ください。

Discovery API を使用すると、実行全体または特定の時点におけるデータパイプラインの健全性とプロジェクトの状態を評価できます。dbt Labs はこの API 用にデフォルトの [GraphQL エクスプローラー](https://metadata.cloud.getdbt.com/graphql) を提供しており、クエリの実行とスキーマの参照が可能です。また、任意の GraphQL クライアントを使用して API をクエリすることもできます。

GraphQL は API 内のデータを記述するため、GraphQL エクスプローラーに表示されるスキーマは、クエリに使用できるグラフとフィールドを正確に表します。

<Snippet path="metadata-api-prerequisites" />

## 承認

現在、リクエストの承認は[サービストークン](/docs/dbt-cloud-apis/service-tokens)を使用して行われます。<Constant name="cloud" /> 管理者ユーザーは、Discovery APIに対して特定のクエリを実行する権限を持つメタデータのみのサービストークンを生成できます。

トークンを作成したら、<Constant name="cloud" /> Discovery APIへのリクエストのAuthorizationヘッダーで使用できます。Authorizationヘッダーには必ずTokenプレフィックスを含めてください。そうしないと、リクエストは`401 Unauthorized`エラーで失敗します。Authorizationヘッダーでは、`Token`の代わりに`Bearer`を使用できます。どちらの構文も同じです。

## Discovery API にアクセスする

1. リクエストを承認するための [サービスアカウントトークン](/docs/dbt-cloud-apis/service-tokens) を作成します。<Constant name="cloud" /> 管理者ユーザーは、メタデータのみのサービストークンを生成できます。このトークンを使用して、Discovery API に対して特定のクエリを実行し、リクエストを承認できます。

2. [Discovery API エンドポイント](#discovery-api-endpoints) テーブルから、使用する API URL を見つけます。

3. 具体的なクエリポイントについては、[スキーマドキュメント](/docs/dbt-cloud-apis/discovery-schema-job) を参照してください。

## HTTP リクエストを使用したクエリの実行

クエリを実行するには、Discovery API に `POST` リクエストを送信します。以下の部分を必ず置き換えてください。
* `YOUR_API_URL` を、リージョンとプランに応じた適切な [Discovery API エンドポイント](#discovery-api-endpoints) に置き換えます。
* Authorization ヘッダーの `YOUR_TOKEN` を、実際の API トークンに置き換えます。トークンプレフィックスを必ず含めてください。
* `QUERY_BODY` を GraphQL クエリに置き換えます。例: `{ "query": "<クエリテキスト>", "variables": "<json 内の変数>" }`
* `VARIABLES` を、ジョブ ID やフィルターなどの GraphQL クエリ変数のディクショナリに置き換えます。
* `ENDPOINT` を、環境などのクエリ対象のエンドポイントに置き換えます。

  ```shell
  curl 'YOUR_API_URL' \
    -H 'authorization: Bearer YOUR_TOKEN' \
    -H 'content-type: application/json'
    -X POST
    --data QUERY_BODY
  ```

Python の例:

```python
response = requests.post(
    'YOUR_API_URL',
    headers={"authorization": "Bearer "+YOUR_TOKEN, "content-type": "application/json"},
    json={"query": QUERY_BODY, "variables": VARIABLES}
)

metadata = response.json()['data'][ENDPOINT]
```

すべてのクエリには環境IDまたはジョブIDが必要です。IDは、<Constant name="cloud" /> URLまたはAdmin APIを使用して取得できます。

このページには、いくつかの分かりやすいクエリ例が掲載されています。その他の例については、[Discovery APIのユースケースと例](/docs/dbt-cloud-apis/discovery-use-cases-and-examples)をご覧ください。

## Discovery API エンドポイント

以下は、Discovery API にアクセスするためのエンドポイントです。ご利用のリージョンとプランに適したエンドポイントをご利用ください。

| Deployment type |	Discovery API URL |
| --------------- | ------------------- |
| North America multi-tenant	|	https://metadata.cloud.getdbt.com/graphql |
| EMEA multi-tenant	|	https://metadata.emea.dbt.com/graphql |
| APAC multi-tenant	|	https://metadata.au.dbt.com/graphql |
| Multi-cell	| `https://YOUR_ACCOUNT_PREFIX.metadata.REGION.dbt.com/graphql`<br /><br />  Replace `YOUR_ACCOUNT_PREFIX` with your specific account identifier and `REGION` with your location, which could be `us1.dbt.com`. |<br />
| Single-tenant | `https://metadata.YOUR_ACCESS_URL/graphql`<br /><br />  Replace `YOUR_ACCESS_URL` with your specific account prefix with the appropriate [Access URL](/docs/cloud/about-cloud/access-regions-ip-addresses) for your region and plan.|

## 合理的な使用

Discovery (GraphQL) API の使用には、メタデータ プラットフォームのパフォーマンスと安定性を維持し、不正使用を防止するため、リクエスト レートとレスポンス サイズに制限があります。

ジョブレベルのエンドポイントには、クエリの複雑さの制限があります。ネストされたノード (親など)、コード (rawCode など)、およびカタログ列は、最も複雑であるとみなされます。過度に複雑なクエリは、必要なフィールドのみを含む個別のクエリに分割する必要があります。dbt Labs は、<Constant name="cloud" /> プロジェクトの最新の記述メタデータと結果メタデータを取得するために、ほとんどのユースケースで環境エンドポイントを使用することを推奨しています。

## 保持期間の制限
Discovery API を使用すると、過去 2 か月間のデータをクエリできます。たとえば、今日が 4 月 1 日の場合、2 月 1 日まで遡ってデータをクエリできます。

## GraphQL エクスプローラーでクエリを実行する

[GraphQL API エクスプローラー](https://metadata.cloud.getdbt.com/graphql) でアドホッククエリを直接実行し、左側のドキュメントエクスプローラーを使用してすべてのノードとフィールドを確認できます。

GraphQL の設定と認証情報については、[Apollo エクスプローラーのドキュメント](https://www.apollographql.com/docs/graphos/explorer/explorer) を参照してください。

1. [GraphQL API エクスプローラー](https://metadata.cloud.getdbt.com/graphql) にアクセスし、クエリを実行するフィールドを選択します。

2. エクスプローラーの下部にある [**変数**] を選択し、`null` フィールドを固有の値に置き換えます。

3. `YOUR_TOKEN` を使用した Bearer 認証を使用して [認証](https://www.apollographql.com/docs/graphos/explorer/connecting-authenticating#authentication) します。エクスプローラーの下部にある [**ヘッダー**] を選択し、[**+新しいヘッダー**] を選択します。

4. [**ヘッダーキー**] ドロップダウンリストから [**Authorization**] を選択し、[**値**] フィールドに Bearer 認証トークンを入力します。トークンプレフィックスを含めることを忘れないでください。ヘッダーキーは、`{"Authorization": "Bearer <YOUR_TOKEN>}` という形式である必要があります。

<!-- TODO: Screenshot needs to be replaced with new one. If we want to show model historical runs, show `environment { applied { modelHistoricalRuns } }` -->
<!-- However we can choose to leave this be, since the important info from the screenshot is to show how the GraphQL API canbe used -- the content (request and response) doesn't matter too much` -->

<br />

<Lightbox src="/img/docs/dbt-cloud/discovery-api/graphql_header.jpg" width="85%" title="Enter the header key and Bearer auth token values"/>

1. **操作**エディタの右上（クエリの右側）にある青いクエリボタンをクリックしてクエリを実行します。エクスプローラーの右側にクエリ成功のレスポンスが表示されます。

<!-- TODO: Screenshot needs to be replaced with new one. If we want to show model historical runs, show `environment { applied { modelHistoricalRuns } }` -->
<!-- However we can choose to leave this be, since the important info from the screenshot is to show how the GraphQL API canbe used -- the content (request and response) doesn't matter too much` -->

<Lightbox src="/img/docs/dbt-cloud/discovery-api/graphql.jpg" width="85%" title="Run queries using the Apollo Server GraphQL explorer"/>

### フラグメント

[`... on`](https://www.apollographql.com/docs/react/data/fragments/) 表記を使用して、系統を横断してクエリを実行し、特定のノードタイプから結果を取得します。

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(first: $first, filter: { uniqueIds: "MODEL.PROJECT.MODEL_NAME" }) {
        edges {
          node {
            name
            ancestors(types: [Model, Source, Seed, Snapshot]) {
              ... on ModelAppliedStateNestedNode {
                name
                resourceType
                materializedType
                executionInfo {
                  executeCompletedAt
                }
              }
              ... on SourceAppliedStateNestedNode {
                sourceName
                name
                resourceType
                freshness {
                  maxLoadedAt
                }
              }
              ... on SnapshotAppliedStateNestedNode {
                name
                resourceType
                executionInfo {
                  executeCompletedAt
                }
              }
              ... on SeedAppliedStateNestedNode {
                name
                resourceType
                executionInfo {
                  executeCompletedAt
                }
              }
            }
          }
        }
      }
    }
  }
}
```

### ページネーション

大規模なデータセットをクエリすると、API パイプライン内の複数の関数のパフォーマンスに影響を与える可能性があります。ページネーションは、小さなデータセットを一度に 1 ページずつ返すことで、この負担を軽減します。これは、データセットの特定の部分、またはデータセット全体を少しずつ返してパフォーマンスを向上させる場合に便利です。<Constant name="cloud" /> はカーソルベースのページネーションを利用するため、常に変化するデータのページを簡単に返すことができます。

`PageInfo` オブジェクトを使用して、ページに関する情報を取得します。使用可能なフィールドは次のとおりです。

- `startCursor` 文字列型 - `edge` の最初の `node` に対応します。
- `endCursor` 文字列型 - `edge` の最後の `node` に対応します。
- `hasNextPage` ブール型 - 返された結果の後にさらに `node` があるかどうかを示します。

クエリ作成時に使用できる接続変数は以下のとおりです。

- `first` 整数型 - 各ページの最初の n 個の `nodes`（最大 500 個）を返します。
- `after` 文字列型 - カーソルを設定して、次の `nodes` を取得します。`after` 変数には、前のページの `endCursor` で定義されたオブジェクト ID を設定することをお勧めします。

以下は、変数で指定されたオブジェクト ID の `後` にある最初の 500 個のモデルを返す例です。`PageInfo` オブジェクトは、カーソルの開始位置、終了位置、および次ページの有無を示すオブジェクト ID を返します。

<!-- TODO: Update screenshot to use `$environmentId: BigInt!, or remove it` -->
<!-- However we can choose to leave this be, since the important info from the screenshot is to show how the GraphQL API canbe used -- the content (request and response) doesn't matter too much` -->

<Lightbox src="/img/Paginate.png" width="75%" title="Example of pagination"/>

以下は `PageInfo` オブジェクトのコード例です。

```graphql
pageInfo {
  startCursor
  endCursor
  hasNextPage
}
totalCount # Total number of records across all pages
```

### フィルター

フィルターを使用すると、API クエリの結果を絞り込むことができます。失敗したモデルやテストのみをクエリして返したい場合や、実行に時間がかかりすぎるモデルを見つけたい場合は、[`executionTime`](/docs/dbt-cloud-apis/discovery-schema-job-models#fields)、[`runElapsedTime`](/docs/dbt-cloud-apis/discovery-schema-job-models#fields)、[`status`](/docs/dbt-cloud-apis/discovery-schema-job-models#fields) などの実行詳細を取得できます。これにより、データチームはモデルのパフォーマンスを監視し、ボトルネックを特定し、データパイプライン全体を最適化できます。

以下は、`lastRunStatus` で成功したモデルの結果をフィルターする例です。

<Lightbox src="/img/Filtering.png" width="75%" title="Example of filtering"/>

以下は、前回の実行でエラーが発生したモデルと失敗したテストをフィルタリングする例です。

<!-- TODO: Update screenshot to use `$environmentId: BigInt!, or remove it` -->
<!-- However we can choose to leave this be, since the important info from the screenshot is to show how the GraphQL API canbe used -- the content (request and response) doesn't matter too much` -->

```graphql
query ModelsAndTests($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(first: $first, filter: { lastRunStatus: error }) {
        edges {
          node {
            name
            executionInfo {
              lastRunId
            }
          }
        }
      }
      tests(first: $first, filter: { status: "fail" }) {
        edges {
          node {
            name
            executionInfo {
              lastRunId
            }
          }
        }
      }
    }
  }
}
```

## 関連コンテンツ

- [Discovery API のユースケースと例](/docs/dbt-cloud-apis/discovery-use-cases-and-examples)
- [スキーマ](/docs/dbt-cloud-apis/discovery-schema-job)