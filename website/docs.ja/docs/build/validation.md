---
title: Validations
id: validation
description: "The Semantic Layer, powered by MetricFlow, has three types of built-in validations, including Parsing Validation, Semantic Validation, and Data Warehouse validation, which are performed in a sequential and blocking manner."
sidebar_label: "Validations"
tags: [Metrics, Semantic Layer]
---

検証とは、システムまたは構成が期待される要件または制約を満たしているかどうかを確認するプロセスを指します。
MetricFlow を搭載したセマンティックレイヤーには、[解析](#parsing)、[セマンティック](#semantic)、[データプラットフォーム](#data-platform) という 3 つの組み込み検証があります。

これらの検証により、構成ファイルが期待されるスキーマに準拠していること、セマンティックグラフが制約に違反していないこと、そしてグラフ内のセマンティック定義が物理テーブルに存在することが保証され、効果的なデータガバナンスがサポートされます。

これらの 3 つの検証ステップは順番に実行され、次のステップに進む前に必ず成功する必要があります。

このトピックについてさらに詳しく知りたい方は、検証を処理するコードを [こちら](https://github.com/dbt-labs/dbt-semantic-interfaces/tree/main/dbt_semantic_interfaces/validations) で参照できます。

## 検証コマンド

以下の[MetricFlow コマンド](/docs/build/metricflow-commands)を使用して、dbt Cloud またはコマンドラインから検証を実行できます。
dbt Cloud では、IDE または CLI で `dbt sl validate-configs` を実行するには開発者認証情報、CI で実行するにはデプロイメント認証情報が必要です。

```bash
dbt sl validate # dbt Cloud users
mf validate-configs # dbt Core users
```

## 解析

この検証ステップでは、設定ファイルが各セマンティックグラフオブジェクトに定義されたスキーマに準拠し、正常に解析できることを確認します。以下のコアオブジェクトのスキーマを検証します。

* Semantic models
* Identifiers
* Measures
* Dimensions
* Metrics

## セマンティック構文

この構文検証ステップは、セマンティックグラフを構築した後に実行されます。
MetricFlow を活用したセマンティックレイヤーは、セマンティックグラフが制約に違反していないことを確認するための一連のテストを実行します。

例えば、メジャー名が一意であるか、マテリアライズで参照されているメトリクスが存在するかを確認します。
現在、チェックするセマンティックルールは以下のとおりです。

1. メジャーを含むセマンティックモデルに有効な時間ディメンションがあることを確認する
2. 各セマンティックモデルに定義されているプラ​​イマリ識別子が1つだけであることを確認する
3. ディメンションの一貫性を確認する
4. セマンティックモデル内のメジャーが一意であることを確認する
5. メトリクス内のメジャーが有効であることを確認する
7. 累積メトリクスが適切に構成されていることを確認する

## データプラットフォーム

このタイプの検証では、セマンティックグラフ内のセマンティック定義が、基盤となる物理テーブルに存在するかどうかを確認します。
これをテストするために、データプラットフォームに対してクエリを実行し、セマンティックモデル、ディメンション、およびメトリック用に生成されたSQLが実行されることを確認します。
以下のチェックを実行します:

* メジャーとディメンションが存在すること
* データソースの基盤となるテーブルが存在すること
* メトリック用に生成されたSQLが実行されること

CIジョブで（セマンティックレイヤーに対して）セマンティック検証を実行することで、dbtモデルに加えられたコード変更がこれらのメトリックに悪影響を与えないことを保証できます。
詳細については、[CIにおけるセマンティック検証](/docs/deploy/ci-jobs#semantic-validations-in-ci)を参照してください。
