---
title: "Discovery API のユースケースと例"
sidebar_label: "Uses and examples"
---

Discovery API を使用すると、<Constant name="cloud" /> 内のメタデータをクエリして、dbt デプロイメントとそのデプロイメントで生成されるデータの詳細を把握し、分析して改善に役立てることができます。

この API は、ビジネス上の疑問に対する答えを得るためにさまざまな方法で使用できます。以下では、API の用途をいくつか紹介し、この API がどのような疑問の解決に役立つかをご紹介します。

| Use case | Outcome | <div style={{width:'400px'}}>Example questions</div> |
| --- | --- | --- |
| [Performance](#performance) | パイプライン実行の非効率性を特定して、インフラストラクチャ コストを削減し、適時性を向上させます。 | <ul><li>各モデルの最新のステータスは何ですか?</li> <li>このモデルを実行する必要がありますか?</li><li>DAG の実行にはどのくらいの時間がかかりましたか?</li> </ul>|
| [Quality](#quality) | データ ソースの鮮度とテスト結果を監視して問題を解決し、データの信頼性を高めます。 | <ul><li>データ ソースはどれくらい新鮮ですか?</li><li>どのテストとモデルが失敗しましたか?</li><li>プロジェクトのテスト範囲はどの程度ですか?</li></ul> |
| [Discovery](#discovery) | 豊富なコンテキストとメタデータを使用して、関連するデータセットとセマンティック ノードを見つけて理解します。 | <ul><li>これらのテーブルと列は何を意味しますか?</li><li>完全なデータ系統とは何ですか?</li><li>どのメトリックをクエリできますか?</li> </ul> |
| [Governance](#governance) | データ開発を監査し、チーム内およびチーム間のコラボレーションを促進します。 | <ul><li>このモデルの責任者は誰ですか?</li><li>モデルの所有者に連絡するにはどうすればよいですか?</li><li>このモデルを使用できるのは誰ですか?</li></ul>|
| [Development](#development) | データセットの変更と使用状況を理解し、影響度を測定してプロジェクト定義に役立てます。 | <ul><li>このメトリックは BI ツールでどのように使用されますか?</li><li>どのノードがこのデータ ソースに依存していますか?</li><li>モデルはどのように変更されましたか?どのような影響がありますか?</li> </ul>|

## パフォーマンス

Discovery API を使用すると、パイプライン実行における非効率性を特定し、インフラストラクチャコストを削減し、タイムリーさを向上させることができます。以下に、実行可能な質問とクエリの例を示します。

パフォーマンス向上のユースケースでは、通常、`environment`、`modelByEnvironment`、またはジョブレベルのエンドポイントを使用して、DAG の任意の部分（モデルなど）の履歴または最新の適用状態をクエリします。

### 各モデルの実行にはどれくらいの時間がかかりましたか？

dbt 実行中にモデル（テーブル）の構築とテストの実行にかかる時間を把握しておくことは有益です。モデルの構築時間が長くなると、インフラストラクチャのコストが増加し、関係者に最新のデータが届くのが遅くなります。このような分析は、オブザーバビリティツールやノートブックなどのアドホッククエリで実行できます。

<Lightbox src="/img/docs/dbt-cloud/discovery-api/model-timing.png" width="200%" title="Model timing visualization in dbt"/>

<details>
<summary>コード付きクエリの例</summary>

データチームは、`executionTime` や `runElapsedTime` などの実行詳細を取得することで、モデルのパフォーマンスを監視し、ボトルネックを特定し、データパイプライン全体を最適化できます。

1. 最新の状態を示す環境レベル API を使用して、実行されたすべてのモデルとその実行時間のリストを取得します。次に、`executionTime` の降順でモデルを並べ替えます。

```graphql
query AppliedModels($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(first: $first) {
        edges {
          node {
            name
            uniqueId
            materializedType
            executionInfo {
              lastSuccessRunId
              executionTime
              executeStartedAt
            }
          }
        }
      }
    }
  }
}
```

2. 最も実行時間が長いモデルの直近20回の実行結果を取得します。実行全体にわたるモデルの結果を確認するか、ジョブ/実行またはコミット自体に移動してさらに調査することもできます。

```graphql
query ModelHistoricalRuns(
  $environmentId: BigInt!
  $uniqueId: String
  $lastRunCount: Int
) {
  environment(id: $environmentId) {
    applied {
      modelHistoricalRuns(
        uniqueId: $uniqueId
        lastRunCount: $lastRunCount
      ) {
        name
        runId
        runElapsedTime
        runGeneratedAt
        executionTime
        executeStartedAt
        executeCompletedAt
        status
      }
    }
  }
}
```

3. クエリ結果を使用して、最も長く実行されているモデルの履歴実行時間と実行時間の傾向のグラフをプロットします。

<!-- TODO: TEST THIS PYTHON CODE WORKS WITH NEW API AND DOCS! -->
```python
# Import libraries
import os
import matplotlib.pyplot as plt
import pandas as pd
import requests

# Set API key
auth_token = *[SERVICE_TOKEN_HERE]*

# Query the API
def query_discovery_api(auth_token, gql_query, variables):
    response = requests.post('https://metadata.cloud.getdbt.com/graphql',
        headers={"authorization": "Bearer "+auth_token, "content-type": "application/json"},
        json={"query": gql_query, "variables": variables})
    data = response.json()['data']

    return data

# Get the latest run metadata for all models
models_latest_metadata = query_discovery_api(auth_token, query_one, variables_query_one)['environment']

# Convert to dataframe
models_df = pd.DataFrame([x['node'] for x in models_latest_metadata['applied']['models']['edges']])

# Unnest the executionInfo column
models_df = pd.concat([models_df.drop(['executionInfo'], axis=1), models_df['executionInfo'].apply(pd.Series)], axis=1)

# Sort the models by execution time
models_df_sorted = models_df.sort_values('executionTime', ascending=False)

print(models_df_sorted)

# Get the uniqueId of the longest running model
longest_running_model = models_df_sorted.iloc[0]['uniqueId']

# Define second query variables
variables_query_two = {
    "environmentId": *[ENVR_ID_HERE]*
    "lastRunCount": 10,
    "uniqueId": longest_running_model
}

# Get the historical run metadata for the longest running model
model_historical_metadata = query_discovery_api(auth_token, query_two, variables_query_two)['environment']['applied']['modelHistoricalRuns']

# Convert to dataframe
model_df = pd.DataFrame(model_historical_metadata)

# Filter dataframe to only successful runs
model_df = model_df[model_df['status'] == 'success']

# Convert the runGeneratedAt, executeStartedAt, and executeCompletedAt columns to datetime
model_df['runGeneratedAt'] = pd.to_datetime(model_df['runGeneratedAt'])
model_df['executeStartedAt'] = pd.to_datetime(model_df['executeStartedAt'])
model_df['executeCompletedAt'] = pd.to_datetime(model_df['executeCompletedAt'])

# Plot the runElapsedTime over time
plt.plot(model_df['runGeneratedAt'], model_df['runElapsedTime'])
plt.title('Run Elapsed Time')
plt.show()

# # Plot the executionTime over time
plt.plot(model_df['executeStartedAt'], model_df['executionTime'])
plt.title(model_df['name'].iloc[0]+" Execution Time")
plt.show()
```

プロット例:

<Lightbox src="/img/docs/dbt-cloud/discovery-api/plot-of-runelapsedtime.png" width="80%" title="The plot of runElapsedTime over time"/>


<Lightbox src="/img/docs/dbt-cloud/discovery-api/plot-of-executiontime.png" width="80%" title="The plot of executionTime over time"/>

</details>

### 各モデルの最新の状態はどうなっていますか？

Discovery API は、モデルの適用状態と、その状態に至るまでの経緯に関する情報を提供します。`environment` エンドポイントから最新の実行と最新の成功した実行（実行）のステータス情報を取得できます。また、ジョブベースおよび `modelByEnvironment` エンドポイントを使用して、過去の実行履歴を詳しく調べることができます。

<details>
<summary>クエリ例</summary>

API は、データベースからの最新の実行と最新の成功した実行の両方について、完全な識別子情報 (`database.schema.alias`) と `executionInfo` を返します:

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(first: $first) {
        edges {
          node {
            uniqueId
            compiledCode
            database
            schema
            alias
            materializedType
            executionInfo {
              executeCompletedAt
              lastJobDefinitionId
              lastRunGeneratedAt
              lastRunId
              lastRunStatus
              lastRunError
              lastSuccessJobDefinitionId
              runGeneratedAt
              lastSuccessRunId
            }
          }
        }
      }
    }
  }
}
```

</details>

### ジョブ実行で何が起こりましたか？

ジョブレベルでメタデータをクエリして、特定の実行結果を確認できます。これは、デプロイメントパフォーマンスの履歴分析や特定のジョブの最適化に役立ちます。

<details>
<summary>クエリ例</summary>

Deprecated example:
```graphql
query ($jobId: Int!, $runId: Int!) {
  models(jobId: $jobId, runId: $runId) {
    name
    status
    tests {
      name
      status
    }
  }
}
```

New example:

```graphql
query ($jobId: BigInt!, $runId: BigInt!) {
  job(id: $jobId, runId: $runId) {
    models {
      name
      status
      tests {
        name
        status
      }
    }
  }
}
```

</details>

### 前回の実行以降に何が変更されましたか？
不要な実行は、インフラストラクチャコストの増加とデータチームとそのシステムの負荷増大につながります。モデルがビューであり、前回の実行以降にコード変更がない場合、またはテーブル/増分モデルであり、前回の実行以降にコード変更がなく、ソースデータが前回の実行以降に更新されていない場合は、モデルを実行する必要はありません。

<details>
<summary>クエリ例</summary>

API を使用すると、定義と適用された状態の間の `rawCode` を比較し、モデルの `materializedType` に基づいて、ソースが最後にロードされた時刻 (モデルの `executeCompletedAt` に対するソースの `maxLoadedAt`) を確認できます。


```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(
        first: $first
        filter: { uniqueIds: "MODEL.PROJECT.MODEL_NAME" }
      ) {
        edges {
          node {
            rawCode
            ancestors(types: [Source]) {
              ... on SourceAppliedStateNestedNode {
                freshness {
                  maxLoadedAt
                }
              }
            }
            executionInfo {
              runGeneratedAt
              executeCompletedAt
            }
            materializedType
          }
        }
      }
    }
    definition {
      models(
        first: $first
        filter: { uniqueIds: "MODEL.PROJECT.MODEL_NAME" }
      ) {
        edges {
          node {
            rawCode
            runGeneratedAt
            materializedType
          }
        }
      }
    }
  }
}
```

</details>

## 品質

Discovery API を使用すると、データソースの鮮度とテスト結果を監視して問題を診断・解決し、データの信頼性を高めることができます。[Webhook](/docs/deploy/webhooks) と併用することで、問題の検出、調査、アラート通知にも役立ちます。以下に、API が回答に役立つ質問の例を示します。また、実行可能な質問とクエリの例も示します。

品質ユースケースでは、通常、DAG の上流部分（ソースなど）にある過去の適用状態または最新の適用状態を、`environment` または `environment { applied { modelHistoricalRuns } }` エンドポイントを使用してクエリします。

### 実行に失敗したモデルとテストはどれですか？

最新のステータスでフィルタリングすると、ビルドに失敗したモデルと、最新の実行時に失敗したテストのリストを取得できます。これは、遅延やデータの誤りにつながるデプロイメントの問題を診断する際に役立ちます。

<details>
<summary>コード付きクエリの例</summary>

1. 環境内のすべてのジョブの最新の実行結果を取得し、エラーが発生した/失敗したモデルとテストのみを返します。

```graphql
query ($environmentId: BigInt!, $first: Int!) {
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

2. 頻繁に使用される重要なデータセットなど、特定のモデルの実行履歴とテスト失敗率 (最大 20 回の実行) を確認します。


```graphql
query ($environmentId: BigInt!, $uniqueId: String!, $lastRunCount: Int) {
  environment(id: $environmentId) {
    applied {
      modelHistoricalRuns(uniqueId: $uniqueId, lastRunCount: $lastRunCount) {
        name
        executeStartedAt
        status
        tests {
          name
          status
        }
      }
    }
  }
}
```

3. 実行を識別し、失敗/エラー率の履歴傾向をプロットします。


</details>


### モデルが使用するデータはいつ最後に更新されましたか？

特定のモデル、またはプロジェクト内のすべてのモデルの最新の実行に関するメタデータを取得できます。例えば、特定のモデルに入力する各モデルまたはスナップショットの最終実行日時、あるいはソースまたはシードの最終ロード日時を調べることで、データの鮮度を測定できます。

<details>
<summary>コード付きクエリの例</summary>


```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(
        first: $first
        filter: { uniqueIds: "MODEL.PROJECT.MODEL_NAME" }
      ) {
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

<!-- TODO: TEST THIS PYTHON CODE WORKS WITH NEW API AND DOCS! -->
```python
# Extract graph nodes from response
def extract_nodes(data):
    models = []
    sources = []
    groups = []
    for model_edge in data["applied"]["models"]["edges"]:
        models.append(model_edge["node"])
    for source_edge in data["applied"]["sources"]["edges"]:
        sources.append(source_edge["node"])
    for group_edge in data["definition"]["groups"]["edges"]:
        groups.append(group_edge["node"])
    models_df = pd.DataFrame(models)
    sources_df = pd.DataFrame(sources)
    groups_df = pd.DataFrame(groups)

    return models_df, sources_df, groups_df

# Construct a lineage graph with freshness info
def create_freshness_graph(models_df, sources_df):
    G = nx.DiGraph()
    current_time = datetime.now(timezone.utc)
    for _, model in models_df.iterrows():
        max_freshness = pd.Timedelta.min
        if "meta" in models_df.columns:
          freshness_sla = model["meta"]["freshness_sla"]
        else:
          freshness_sla = None
        if model["executionInfo"]["executeCompletedAt"] is not None:
          model_freshness = current_time - pd.Timestamp(model["executionInfo"]["executeCompletedAt"])
          for ancestor in model["ancestors"]:
              if ancestor["resourceType"] == "SourceAppliedStateNestedNode":
                  ancestor_freshness = current_time - pd.Timestamp(ancestor["freshness"]['maxLoadedAt'])
              elif ancestor["resourceType"] == "ModelAppliedStateNestedNode":
                  ancestor_freshness = current_time - pd.Timestamp(ancestor["executionInfo"]["executeCompletedAt"])

              if ancestor_freshness > max_freshness:
                  max_freshness = ancestor_freshness

          G.add_node(model["uniqueId"], name=model["name"], type="model", max_ancestor_freshness = max_freshness, freshness = model_freshness, freshness_sla=freshness_sla)
    for _, source in sources_df.iterrows():
        if source["maxLoadedAt"] is not None:
          G.add_node(source["uniqueId"], name=source["name"], type="source", freshness=current_time - pd.Timestamp(source["maxLoadedAt"]))
    for _, model in models_df.iterrows():
        for parent in model["parents"]:
            G.add_edge(parent["uniqueId"], model["uniqueId"])

    return G
```

グラフの例:

<Lightbox src="/img/docs/dbt-cloud/discovery-api/lineage-graph-with-freshness-info.png" width="75%" title="A lineage graph with source freshness information"/>

</details>


### データソースは最新ですか？

[ソースの鮮度](/docs/build/sources#source-data-freshness)を確認することで、dbtプロジェクトで読み込まれ、使用されるソースが期待どおりであることを確認できます。APIは、ソースの読み込みに関する最新のメタデータと鮮度チェックの基準に関する情報を提供します。

<Lightbox src="/img/docs/dbt-cloud/discovery-api/source-freshness-page.png" width="75%" title="Source freshness page in dbt"/>

<details>
<summary>クエリ例</summary>

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      sources(
        first: $first
        filter: { freshnessChecked: true, database: "production" }
      ) {
        edges {
          node {
            sourceName
            name
            identifier
            loader
            freshness {
              freshnessJobDefinitionId
              freshnessRunId
              freshnessRunGeneratedAt
              freshnessStatus
              freshnessChecked
              maxLoadedAt
              maxLoadedAtTimeAgoInS
              snapshottedAt
              criteria {
                errorAfter {
                  count
                  period
                }
                warnAfter {
                  count
                  period
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

</details>

### テストの範囲とステータスは？

[テスト](https://docs.getdbt.com/docs/build/tests) は、関係者が高品質なデータをレビューしていることを確認するための重要な手段です。dbt 実行中にテストを実行できます。Discovery API は、特定の環境またはジョブの完全なテスト結果を提供します。これらのテスト結果は、テスト済みの特定のノード（例: モデル）の「子」として表されます。

<details>
<summary>クエリ例</summary>

次の例では、`parents` はテスト対象のノード (コード) であり、`executionInfo` は最新のテスト結果を説明します。

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      tests(first: $first) {
        edges {
          node {
            name
            columnName
            parents {
              name
              resourceType
            }
            executionInfo {
              lastRunStatus
              lastRunError
              executeCompletedAt
              executionTime
            }
          }
        }
      }
    }
  }
}
```

</details>

### このモデルはどのようにコントラクト化され、バージョン管理されていますか？

モデル定義の形状を強制するために、モデルとその列にコントラクトを定義できます。また、モデルのバージョンを指定して、進化の段階を追跡し、適切なバージョンを使用することもできます。

<!-- TODO: The description above is not accurate for the desired query below because only applied models can query catalogs, so the query is changed to `environment.applied`. We need to change the description to refer to the applied state, or do not query `catalog` from the definition state node. -->

<details>
<summary>クエリ例</summary>


```graphql
query {
  environment(id: 123) {
    applied {
      models(first: 100, filter: { access: public }) {
        edges {
          node {
            name
            latestVersion
            contractEnforced
            constraints {
              name
              type
              expression
              columns
            }
            catalog {
              columns {
                name
                type
              }
            }
          }
        }
      }
    }
  }
}
```

</details>

## Discovery

Discovery API を使用すると、豊富なコンテキストとメタデータを持つ関連データセットとセマンティックノードを検索して理解できます。以下に、実行可能な質問とクエリの例を示します。

Discovery のユースケースでは、通常、`environment` エンドポイントを使用して、DAG の下流部分（たとえば、スマート モデルやメトリクス）にある最新の適用状態または定義状態をクエリします。

### このデータセットとその列は何を意味するのでしょうか？

Discovery API にクエリを実行して、データプラットフォーム内のテーブル/ビューを dbt プロジェクトのモデルにマッピングします。次に、YAML ファイルからの説明メタデータや、YAML ファイルとスキーマからのカタログ情報など、その意味に関するメタデータを取得します。

<details>
<summary>クエリ例</summary>

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(
        first: $first
        filter: {
          database: "analytics"
          schema: "prod"
          identifier: "customers"
        }
      ) {
        edges {
          node {
            name
            description
            tags
            meta
            catalog {
              columns {
                name
                description
                type
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

<!-- TODO: Revise this section to use the `environment.definition.lineage` endpoints instead of querying all nodes

### What’s the full data lineage?

Lineage, enabled by the `ref` function, is at the core of dbt. Understanding lineage provides many benefits, such as understanding the structure and relationships of datasets (and metrics) and performing impact-and-root-cause analyses to resolve or present issues given changes to definitions or source data. With the Discovery API, you can construct lineage using the `parents` nodes or its `children` and query the entire upstream lineage using `ancestors`.

<Lightbox src="/img/docs/dbt-cloud/discovery-api/example-dag.png" width="80%" title="Example of a DAG"/>

<details>
<summary>Example query with code</summary>

1. Query all project nodes

```graphql
query Lineage($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    definition {
      sources(first: $first) {
        edges {
          node {
            uniqueId
            name
            resourceType
            children {
              uniqueId
              name
              resourceType
            }
          }
        }
      }
      seeds(first: $first) {
        edges {
          node {
            uniqueId
            name
            resourceType
            children {
              uniqueId
              name
              resourceType
            }
          }
        }
      }
      snapshots(first: $first) {
        edges {
          node {
            uniqueId
            name
            resourceType
            parents {
              uniqueId
              name
              resourceType
            }
            children {
              uniqueId
              name
              resourceType
            }
          }
        }
      }
      models(first: $first) {
        edges {
          node {
            uniqueId
            name
            resourceType
            parents {
              uniqueId
              name
              resourceType
            }
            children {
              uniqueId
              name
              resourceType
            }
          }
        }
      }
      exposures(first: $first) {
        edges {
          node {
            uniqueId
            name
            resourceType
            parents {
              uniqueId
              name
              resourceType
            }
          }
        }
      }
      # metrics and semanticModels coming soon...
    }
  }
}
```

Then, extract the node definitions and create a lineage graph. You can traverse downstream from sources and seeds (adding an edge from each node with children to its children) or iterate through each node’s parents (if it has them). Remember that models, snapshots, and metrics can have parents and children, whereas sources and seeds have only children and exposures only have parents.


2. Extract the node definitions, construct a lineage graph, and plot the graph.

```python
# TODO: TEST THIS PYTHON CODE WORKS WITH NEW API AND DOCS!
import networkx as nx
import os
import matplotlib.pyplot as plt
import pandas as pd
import requests
from collections import defaultdict

# Write Discovery API query
gql_query = """
query Definition($environmentId: BigInt!, $first: Int!) {
*[ADD QUERY HERE]*
}

"""

# Define query variables
variables = {
    "environmentId": *[ADD ENV ID HERE]*,
    "first": 500
}


# Query the API
def query_discovery_api(auth_token, gql_query, variables):
    response = requests.post('https://metadata.cloud.getdbt.com/beta/graphql',
        headers={"authorization": "Bearer "+auth_token, "content-type": "application/json"},
        json={"query": gql_query, "variables": variables})
    data = response.json()['data']['environment']

    return data


# Extract nodes for graph
def extract_node_definitions(api_response):
    nodes = []
    node_types = ["models", "sources", "seeds", "snapshots", "exposures"]  # support for metrics and semanticModels coming soon
    for node_type in node_types:
        if node_type in api_response["definition"]:
            for node_edge in api_response["definition"][node_type]["edges"]:
                node_edge["node"]["type"] = node_type
                nodes.append(node_edge["node"])
    nodes_df = pd.DataFrame(nodes)
		return nodes_df


# Construct the graph
def create_generic_lineage_graph(nodes_df):
    G = nx.DiGraph()
    for _, node in nodes_df.iterrows():
        G.add_node(node["uniqueId"], name=node["name"], type=node["type"])
    for _, node in nodes_df.iterrows():
        if node["type"] not in ["sources", "seeds"]:
          for parent in node["parents"]:
              G.add_edge(parent["uniqueId"], node["uniqueId"])
    return G


# Assign graph layers
def assign_layers(G):
    layers = {}
    layer_counts = defaultdict(int)
    for node in nx.topological_sort(G):
        layer = 0
        for parent in G.predecessors(node):
            layer = max(layers[parent] + 1, layer)
        layers[node] = layer
        layer_counts[layer] += 1
    nx.set_node_attributes(G, layers, "layer")
    return layer_counts


# Plot the lineage graph
def plot_generic_graph(G):
    plt.figure(figsize=(10, 6.5))

    # Assign layers to the nodes
    layer_counts = assign_layers(G)

    # Use the multipartite_layout to create a layered layout
    pos = nx.multipartite_layout(G, subset_key="layer", align='vertical', scale=2)

    # Adjust the y-coordinate of nodes to spread them out
    y_offset = 1.0
    for node, coords in pos.items():
        layer = G.nodes[node]["layer"]
        coords[1] = (coords[1] - 0.5) * (y_offset * layer_counts[layer])

    # Define a color mapping for node types
    type_color_map = {
        "models": "blue",
        "sources": "green",
        "seeds": "lightgreen",
        "snapshots": "lightblue",
        "metrics": "red",
        "exposures": "orange"
    }

    node_colors = [type_color_map[G.nodes[n].get("type")] for n in G.nodes()]
    nx.draw(G, pos, node_color=node_colors, node_shape="s", node_size=3000, bbox=dict(facecolor="white", edgecolor='gray', boxstyle='round,pad=0.1'), edgecolors='Gray', alpha=0.8, with_labels=True, labels={n: G.nodes[n].get('name') for n in G.nodes()}, font_size=11, font_weight="bold")
    plt.axis("off")
    plt.show()


query_response = query_discovery_api(auth_token, gql_query, variables)

nodes_df = extract_node_definitions(query_response)

G = create_generic_lineage_graph(nodes_df)

plot_generic_graph(G)
```

Graph example:

<Lightbox src="/img/docs/dbt-cloud/discovery-api/lineage-graph.png" width="75%" title="A lineage graph"/>


</details>

-->

### どのようなメトリクスが利用可能ですか？

[<Constant name="semantic_layer" />](/docs/build/about-metricflow) を使用してメトリクスを定義およびクエリし、ドキュメント作成目的（データカタログなど）で使用したり、集計計算（SL をクエリしない BI ツールなど）に使用したりできます。

<details>
<summary>クエリ例</summary>

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    definition {
      metrics(first: $first) {
        edges {
          node {
            name
            description
            type
            formula
            filter
            tags
            parents {
              name
              resourceType
            }
          }
        }
      }
    }
  }
}
```

</details>

## ガバナンス

Discovery API を使用すると、データ開発を監査し、チーム内およびチーム間のコラボレーションを促進できます。

ガバナンスのユースケースでは、多くの場合、DAG の下流部分（パブリックモデルなど）にある最新の定義状態を `environment` エンドポイントを使用して照会する傾向があります。

### このモデルの責任者は誰ですか？

各モデルが関連付けられているグループを定義し、表示できます。グループには所有者などの情報が含まれます。これにより、特定のモデルを所有するチームや、そのモデルについて誰に連絡すればよいかを特定できます。

<details>
<summary>クエリ例</summary>

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      models(first: $first, filter: { uniqueIds: ["MODEL.PROJECT.NAME"] }) {
        edges {
          node {
            name
            description
            resourceType
            access
            group
          }
        }
      }
    }
    definition {
      groups(first: $first) {
        edges {
          node {
            name
            resourceType
            models {
              name
            }
            ownerName
            ownerEmail
          }
        }
      }
    }
  }
}
```
</details>

### このモデルは誰が使用できますか？

特定のモデルへのアクセスレベルを指定できる権限をユーザーに付与できます。将来的には、パブリックモデルはAPIのように機能し、プロジェクトの系統を統合し、プロジェクト間の参照を使用してモデルの再利用を可能にする予定です。

<details>
<summary>クエリ例</summary>

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    definition {
      models(first: $first) {
        edges {
          node {
            name
            access
          }
        }
      }
    }
  }
}
```

---

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    definition {
      models(first: $first, filter: { access: public }) {
        edges {
          node {
            name
          }
        }
      }
    }
  }
}
```
</details>

## 開発

Discovery API を使用すると、データセットの変更と使用状況を把握し、影響度を測定してプロジェクト定義に役立てることができます。以下に、実行可能な質問とクエリの例を示します。

開発ユースケースでは、通常、`environment` エンドポイントを使用して、DAG の任意の部分における過去の定義や最新の定義、または適用状態をクエリします。

### このモデルまたはメトリクスは下流のツールでどのように使用されますか？
[エクスポージャー](/docs/build/exposures)は、ダッシュボードやその他の分析ツール、ユースケースでモデルまたはメトリクスが実際にどのように使用されるかを定義する方法を提供します。エクスポージャーの定義をクエリしてプロジェクトノードがどのように使用されているかを確認したり、上流のリネージ結果をクエリしてそこで使用されているデータの状態を把握したりできます。これは、鮮度や品質ステータスタイルなどのユースケースに役立ちます。

<Lightbox src="/img/docs/collaborate/dbt-explorer/data-tile-pass.jpg" width="60%" title="Embed data health tiles in your dashboards to distill trust signals for data consumers." />


<details>
<summary>クエリ例</summary>

以下は、エクスポージャーと、その中で使用されたモデル（最後に実行された日時を含む）を確認する例です。

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      exposures(first: $first) {
        edges {
          node {
            name
            description
            ownerName
            url
            parents {
              name
              resourceType
              ... on ModelAppliedStateNestedNode {
                executionInfo {
                  executeCompletedAt
                  lastRunStatus
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
</details>

### このモデルは時間の経過とともにどのように変化しましたか？

Discovery API は、プロジェクト内のあらゆるリソースに関する履歴情報を提供します。例えば、モデルの形状や内容の変化に応じて、モデルが時間の経過とともにどのように進化したか（最近の実行履歴全体にわたって）を確認できます。

<details>
<summary>クエリ例</summary>

実行間の `compiledCode` または `columns` の違いを確認したり、時間の経過に伴う「おおよそのサイズ」と「行数」の `stats` をプロットしたりします:

```graphql
query (
  $environmentId: BigInt!
  $uniqueId: String!
  $lastRunCount: Int!
  $withCatalog: Boolean!
) {
  environment(id: $environmentId) {
    applied {
      modelHistoricalRuns(
        uniqueId: $uniqueId
        lastRunCount: $lastRunCount
        withCatalog: $withCatalog
      ) {
        name
        compiledCode
        columns {
          name
        }
        stats {
          label
          value
        }
      }
    }
  }
}
```
</details>

### このデータソースに依存するノードはどれですか？

dbt の系統はデータソースから始まります。特定のソースについて、どのノードが子であるかを確認し、下流に反復処理することで依存関係の完全なリストを取得できます。

現在、1世代（直接の親から子として定義）を超えるクエリはサポートされていません。ノードの孫を表示するには、2つのクエリを実行する必要があります。1つはノードとその子を取得するクエリ、もう1つは子ノードとその子を取得するクエリです。

<details>
<summary>クエリ例</summary>

```graphql
query ($environmentId: BigInt!, $first: Int!) {
  environment(id: $environmentId) {
    applied {
      sources(
        first: $first
        filter: { uniqueIds: ["SOURCE_NAME.TABLE_NAME"] }
      ) {
        edges {
          node {
            loader
            children {
              uniqueId
              resourceType
              ... on ModelAppliedStateNestedNode {
                database
                schema
                alias
              }
            }
          }
        }
      }
    }
  }
}
```
</details>

## 関連ドキュメント

- [クエリ検出 API](/docs/dbt-cloud-apis/discovery-querying)
