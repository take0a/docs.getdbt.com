---
title: "Semantic manifest"
id: sl-manifest
description: "セマンティックの manifest.json ファイルについて、またアーティファクトを使用して dbt セマンティック レイヤーに関する分析情報を取得する方法について学習します。"
tags: [Semantic Layer, APIs]
sidebar_label: "Semantic manifest"
pagination_next: null
---

**生成元:** プロジェクトを解析する任意のコマンド。これには、[`deps`](/reference/commands/deps)、[`clean`](/reference/commands/clean)、[`debug`](/reference/commands/debug)、[`init`](/reference/commands/init)を除くすべてのコマンドが含まれます。

dbt は、_セマンティックマニフェスト_ (`semantic_manifest.json`) と呼ばれる [アーティファクト](/reference/artifacts/dbt-artifacts) ファイルを作成します。これは、MetricFlow が dbt セマンティックレイヤーのメトリッククエリを適切に構築および実行するために必要なものです。このアーティファクトには、dbt セマンティックレイヤーに関する包括的な情報が含まれています。これは、MetricFlow との統合ポイントとして機能する内部ファイルです。

dbt Core によって生成されたセマンティックマニフェストを使用して、MetricFlow はデータフロープランをインスタンス化し、セマンティックレイヤーのクエリリクエストから SQL を生成します。これは、データモデルの構造と詳細を理解するのに役立つ貴重なリファレンスです。

[`manifest.json` ファイル](/reference/artifacts/manifest-json) と同様に、`semantic_manifest.json` ファイルも dbt プロジェクトの [ターゲットディレクトリ](/reference/global-configs/json-artifacts) に配置されます。このディレクトリには、dbt がプロジェクトの実行中に生成するさまざまなアーティファクト（コンパイル済みモデルやテストなど）が格納されます。

`semantic_manifest.json` が `manifest.json` と共存する理由は 2 つあります。

- デシリアライゼーション: `dbt-core` と MetricFlow は、データのシリアル化を処理するために異なるライブラリを使用します。
- 効率性とパフォーマンス: MetricFlow と dbt セマンティックレイヤーは、マニフェストから特定のセマンティック詳細を必要とします。 `semantic_manifest.json` に出力される情報を削減することで、プロセスがより効率的になり、`dbt-core` と MetricFlow 間のデータ処理が高速化されます。

## トップレベルキー

セマンティックマニフェストのトップレベルキーは次のとおりです。
- `semantic_models` - エンティティ、ディメンション、メジャーを含むデータの開始点であり、dbt プロジェクト内のモデルに対応します。
- `metrics` - メジャー、制約などを組み合わせて定量的な指標を定義する関数です。
- `project_configuration` - プロジェクト構成に関する情報が含まれます。

### 例

<File name="target/semantic_manifest.json"> 

```json
{
    "semantic_models": [
        {
            "name": "semantic model name",
            "defaults": null,
            "description": "semantic model description",
            "node_relation": {
                "alias": "model alias",
                "schema_name": "model schema",
                "database": "model db",
                "relation_name": "Fully qualified relation name"
            },
            "entities": ["entities in the semantic model"],
            "measures": ["measures in the semantic model"],
            "dimensions": ["dimensions in the semantic model" ],
    "metrics": [
        {
            "name": "name of the metric",
            "description": "metric description",
            "type": "metric type",
            "type_params": {
                "measure": {
                    "name": "name for measure",
                    "filter": "filter for measure",
                    "alias": "alias for measure"
                },
                "numerator": null,
                "denominator": null,
                "expr": null,
                "window": null,
                "grain_to_date": null,
                "metrics": ["metrics used in defining the metric. this is used in derived metrics"],
                "input_measures": []
            },
            "filter": null,
            "metadata": null
        }
    ],
    "project_configuration": {
        "time_spine_table_configurations": [
            {
                "location": "fully qualified table name for timespine",
                "column_name": "date column",
                "grain": "day"
            }
        ],
        "metadata": null,
        "dsi_package_version": {}
    },
    "saved_queries": [
        {
            "name": "name of the saved query",
            "query_params": {
                "metrics": [
                    "metrics used in the saved query"
                ],
                "group_by": [
                    "TimeDimension('model_primary_key__date_column', 'day')",
                    "Dimension('model_primary_key__metric_one')",
                    "Dimension('model__dimension')"
                ],
                "where": null
            },
            "description": "Description of the saved query",
            "metadata": null,
            "label": null,
            "exports": [
                {
                    "name": "saved_query_name",
                    "config": {
                        "export_as": "view",
                        "schema_name": null,
                        "alias": null
                    }
                }
            ]
        }
    ]
}
    ]
}
```

</File>

## 関連ドキュメント

- [dbt セマンティックレイヤー API](/docs/dbt-cloud-apis/sl-api-overview)
- [dbt アーティファクトについて](/reference/artifacts/dbt-artifacts)
