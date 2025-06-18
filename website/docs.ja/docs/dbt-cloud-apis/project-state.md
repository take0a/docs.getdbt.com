---
title: "dbt でのプロジェクトの状態"
---

<Constant name="cloud" /> は、dbt をステートフルにデプロイする方法を提供します。アーティファクトには、メタデータ プラットフォームの [Discovery API](/docs/dbt-cloud-apis/discovery-querying) を介してプログラムでアクセスできます。

Discovery API に `environment` エンドポイントを実装することで、複数の状態の概念が導入されました。Discovery API は、DAG 内のモデル、ソース、その他のノードの最新の状態を返す単一の API エンドポイントを提供します。

単一の [デプロイメント環境](/docs/environments-in-dbt) は、特定の <Constant name="cloud" /> プロジェクトの運用状態を表します。

<Constant name="cloud" /> でクエリできる状態は 2 つあります。

- **適用状態** は、`dbt run` が成功した後にデータ ウェアハウスに存在する状態を指します。モデルのビルドは成功し、ウェアハウス内にテーブルとして存在します。

- **定義状態** は、プロジェクト内に定義されているコード (マニフェスト状態など) に基づいてプロジェクト内に存在するものに依存しますが、必ずしもデータ プラットフォームで実行されているわけではありません (`dbt compile` の結果だけの場合もあります)。

## dbt ノードの定義（論理）状態と適用状態

dbt プロジェクトにおいて、ノードの状態（_definition_）は、SQL ファイルおよび YAML ファイルで定義された構成、変換、および依存関係を表します。これは、データウェアハウス内の他のノードやテーブルとの関係において、ノードがどのように処理されるべきかを示すもので、`dbt build`、`run`、`parse`、または `compile` によって生成される場合があります。この状態は、プロジェクトコードが変更されるたびに変化します。

ノードの_applied state_ は、DAG でノードが正常に実行された後の実際の状態を指します。たとえば、モデルが実行されると、その状態は `dbt run` または `dbt build` を介してデータウェアハウスに適用されます。この状態は、ノードが実行されるたびに変化します。この状態は、変換の結果と、データベースに保存された実際のデータを表します。モデルの場合、これは定義されたロジックに基づくテーブルまたはビューになります。

適用状態には実行情報が含まれます。実行情報には、ノードが適用状態に到達した経緯に関するメタデータ（最新の実行（成功または試行）、開始時刻、ステータス、所要時間など）が含まれます。

Discovery API を使用してモデルの定義状態と適用状態をクエリして比較する方法は次のとおりです:

```graphql
query Compare($environmentId: Int!, $first: Int!) {
	environment(id: $environmentId) {
		definition {
			models(first: $first) {
				edges {
					node {
						name
						rawCode
					}
				}
			}
		}
		applied {
			models(first: $first) {
				edges {
					node {
						name
						rawCode 
						executionInfo {
							executeCompletedAt
						}
					}
				}
			}
		}
	}
}

```

ほとんどの Discovery API の使用例では、実際に実行されて分析できる内容に関係するため、_適用済み状態_ が優先されます。
 
## ノードタイプ別の影響を受ける状態

次の表は、dbt ノードの状態と、Discovery API によって影響を受ける内容を示しています。

| Node                                          | Executed in DAG  | Created by execution | Exists in database | Lineage               | States               |
|-----------------------------------------------|------------------|----------------------|--------------------|-----------------------|----------------------|
| [Analysis](/docs/build/analyses)   	        | No               | No                   | No                 | Upstream            | Definition 	      |
| [Data test](/docs/build/data-tests)           | Yes              | Yes                  | No                 | Upstream              | Applied & definition |
| [Exposure](/docs/build/exposures)             | No               | No                   | No                 | Upstream              | Definition           |
| [Group](/docs/build/groups)                   | No               | No                   | No                 | Downstream            | Definition           |
| [Macro](/docs/build/jinja-macros)             | Yes              | No                   | No                 | N/A                   | Definition           |
| [Metric](/docs/build/metrics-overview)     	| No               | No                   | No                 | Upstream & downstream | Definition           |
| [Model](/docs/build/models)                   | Yes              | Yes                  | Yes                | Upstream & downstream | Applied & definition |
| [Saved queries](/docs/build/saved-queries) <br /> (not in API)  | N/A               | N/A                  |   N/A         | N/A | N/A           |
| [Seed](/docs/build/seeds)                     | Yes              | Yes                  | Yes                | Downstream            | Applied & definition |
| [Semantic model](/docs/build/semantic-models) | No               | No                   | No                 | Upstream & downstream | Definition           |
| [Snapshot](/docs/build/snapshots)             | Yes              | Yes                  | Yes                | Upstream & downstream | Applied & definition |
| [Source](/docs/build/sources)                 | Yes              | No                   | Yes                | Downstream            | Applied & definition |
| [Unit tests](/docs/build/unit-tests)          | Yes              | Yes                  | No                 | Downstream   	       | Definition 	      |


## 状態/メタデータの更新に関する注意事項

今後、Cloud Artifacts は <Constant name="cloud" /> 内の機能/サービスの状態を維持するための情報を提供し、<Constant name="cloud" /> およびその下流のエコシステムの状態にアクセスできるようになります。Cloud Artifacts は現在、最新の本番環境の状態に重点を置いていますが、この重点は今後進化していく予定です。

Discovery API における状態表現には、次のような制限があります。

- プロジェクトの最新の状態を確認するには、デフォルトの本番環境にアクセスする必要があります。
- API は、特定のデプロイ環境で生成された最新のマニフェストから定義を取得しますが、これは必ずしも最新のプロジェクトコードの状態を反映しているとは限りません。
- コンパイルされたコードの結果は、<Constant name="cloud" /> の実行順序や失敗によっては、古くなっている可能性があります。
- カタログ情報は、`docs generate` が最後に実行されたかどうか、またはいつ実行されたかによって、古くなったり、不完全になったりする可能性があります（適用済みの状態）。
- コマンドが最後に実行された時期によっては、ソースの鮮度チェックが古くなっている可能性があります (適用された状態)。これは `build` には含まれません。
