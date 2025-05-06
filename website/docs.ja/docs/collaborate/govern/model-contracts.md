---
title: "Model contracts"
id: model-contracts
sidebar_label: "Model contracts"
description: "Model contracts define a set of parameters validated during transformation"
---

## 関連ドキュメント
* [`contract`](/reference/resource-configs/contract)
* [`columns`](/reference/resource-properties/columns)
* [`constraints`](/reference/resource-properties/constraints)

## なぜコントラクトを定義するのか？

dbt モデルの定義は、SQL の `select` 文を書くのと同じくらい簡単です。クエリを実行すると、選択した列と適用した変換に基づいて、名前と型の列を持つデータセットが自動的に生成されます。

これは迅速かつ反復的な開発には理想的ですが、モデルによっては、返されるデータセットの形状が常に変更されると、他の人やプロセスがそのモデルをクエリする際にリスクが生じます。そのため、モデルの形状を定義する一連の「保証」を事前に定義しておくことをお勧めします。この一連の保証を「コントラクト」と呼びます。モデルの構築中、dbt はモデルの変換によってコントラクトに一致するデータセットが生成されるかどうかを検証します。そうでない場合は、構築に失敗します。

import ModelGovernanceRollback from '/snippets/_model-governance-rollback.md';

<ModelGovernanceRollback />

## コントラクトはどこでサポートされていますか？

現在、モデル コントラクトは次のモデルでサポートされています。
- SQL モデル。
- 次のいずれかとしてマテリアライズされたモデル。
  - `table`
  - `view` - ビューは列名とデータ型を限定的にサポートしますが、`constraints` はサポートしていません。
  - `incremental` - `on_schema_change: append_new_columns` または `on_schema_change: fail` が指定されているもの。
- 特定のデータ プラットフォーム。ただし、サポートおよび適用される `constraints` はプラットフォームによって異なります。

モデル コントラクトは次のモデルではサポートされていません。
- Python モデル。
- `materialized view` または `ephemeral` マテリアライズされた SQL モデル。
- カスタム マテリアライズ（作成者によって追加されていない場合）。
- BigQuery で再帰的な <Term id="cte" /> を含むモデル。
- `sources`、`seeds`、`snapshots` などのその他のリソース タイプ。

## コントラクトの定義方法

次のようなクエリを持つモデルがあるとします:

<File name="models/marts/dim_customers.sql">

```sql
-- lots of SQL

final as (

    select
        customer_id,
        customer_name,
        -- ... many more ...
    from ...

)

select * from final
```

</File>

モデルのコントラクトを適用するには、`contract` 設定で `enforced: true` を設定します。

適用する場合、コントラクトにはすべての列の `name` と `data_type` を含める必要があります（`data_type` はデータプラットフォームが理解できるものと一致します）。

モデルが `table` または `incremental` としてマテリアライズされている場合、データプラットフォームによっては、`not_null`（null 値が 0 個含まれる）などの追加の [制約](/reference/resource-properties/constraints) をオプションで指定できます。

<File name="models/marts/customers.yml">

```yaml
models:
  - name: dim_customers
    config:
      contract:
        enforced: true
    columns:
      - name: customer_id
        data_type: int
        constraints:
          - type: not_null
      - name: customer_name
        data_type: string
      ...
```

</File>

定義されたコントラクトを使用してモデルを構築する場合、dbt は 2 つの処理を異なる方法で実行します。
1. dbt は「プリフライト」チェックを実行し、モデルのクエリが定義済みのものと一致する名前とデータ型の列セットを返すことを確認します。このチェックは、モデル (SQL) または YAML 仕様で指定された列の順序とは無関係です。
2. dbt は、データ プラットフォームに送信する DDL ステートメントに列名、データ型、および制約を含めます。これは、モデルのテーブルの構築または更新中に適用されます。また、dbt モデルではなくコントラクトに従って列を順序付けます。

## プラットフォーム制約のサポート

各プラットフォームにおける [制約](/reference/resource-properties/constraints) のサポートに関する詳細については、アダプタ固有のタブを選択してください。制約は、定義可能性とプラットフォームの適用に基づいて、以下の3つのカテゴリに分類されます。

- **定義可能かつ適用** &mdash; 制約に違反する場合、モデルはビルドされません。
- **定義可能かつ適用なし** &mdash; プラットフォームは制約の種類の指定をサポートしていますが、モデルのビルドが制約に違反する場合でも、モデルはビルドできます。この制約はメタデータ作成のみを目的としています。このアプローチは、厳格なルール適用が一般的であるトランザクションデータベースよりも、クラウドデータウェアハウスでより一般的です。
- **定義不可かつ適用なし** &mdash; プラットフォームに対して制約の種類を指定することはできません。



<Tabs>

<TabItem value="Redshift" label="Redshift">

| Constraint type | Definable       | Enforced         |
|:----------------|:-------------:|:------------------:|
| not_null        | ✅ | ✅ |
| primary_key     | ✅ | ❌ |
| foreign_key     | ✅ | ❌ |
| unique          | ✅ | ❌ |
| check           | ❌ | ❌ |

</TabItem>
<TabItem value="Snowflake" label="Snowflake">

| Constraint type | Definable     | Enforced |
|:----------------|:-------------:|:---------------------:|
| not_null        | ✅  | ✅ |
| primary_key     | ✅  | ❌ |
| foreign_key     | ✅  | ❌ |
| unique          | ✅  | ❌ |
| check           | ❌  | ❌ |

</TabItem>
<TabItem value="BigQuery" label="BigQuery">

| Constraint type | Definable     | Enforced |
|:-----------------|:-------------:|:---------------------:|
| not_null        | ✅ | ✅  |
| primary_key     | ✅ | ❌  |
| foreign_key     | ✅ | ❌  |
| unique          | ❌ | ❌  |
| check           | ❌ | ❌  |

</TabItem>
<TabItem value="Postgres" label="Postgres">

| Constraint type | Definable     | Enforced |
|:----------------|:-------------:|:--------------------:|
| not_null        | ✅  |	✅  |
| primary_key     | ✅  |	✅  |
| foreign_key     | ✅  |	✅  |
| unique          | ✅  |	✅  |
| check           | ✅  |	✅  |

</TabItem>
<TabItem value="Spark" label="Spark">

現在、`not_null` 制約と `check` 制約は、モデルがビルドされた後にのみ適用されます。このプラットフォームの制限により、dbt はこれらの制約を定義可能ではあるものの適用されないものと見なします。つまり、ビルド時に適用できないため、これらの制約は _model contract_ の一部ではないということです。この表は、機能の進化に伴って変更されます。

| Constraint type | Definable    | Enforced |
|:----------------|:------------:|:---------------------:|
| not_null        |	✅  | ❌ |
| primary_key     |	✅  | ❌ |
| foreign_key     |	✅  | ❌ |
| unique          |	✅  | ❌ |
| check           |	✅  | ❌ |

</TabItem>
<TabItem value="Databricks" label="Databricks">

現在、`not_null` 制約と `check` 制約は、モデルがビルドされた後にのみ適用されます。このプラットフォームの制限により、dbt はこれらの制約を定義可能ではあるものの適用されないものと見なします。つまり、ビルド時に適用できないため、これらの制約は_model contract_ の一部ではないということです。この表は、機能の進化に伴って変更されます。

| Constraint type | Definable     | Enforced |
|:----------------|:-------------:|:---------------------:|
| not_null        |	✅  | ❌ |
| primary_key     | ✅  | ❌ |
| foreign_key     |	✅  | ❌ |
| unique          |	✅  | ❌ |
| check           |	✅  | ❌ |

</TabItem>
<TabItem value="Athena" label="Athena">

| Constraint type | Definable     | Enforced |
|:----------------|:-------------:|:---------------------:|
| not_null        |	❌  | ❌ |
| primary_key     | ❌  | ❌ |
| foreign_key     |	❌  | ❌ |
| unique          |	❌  | ❌ |
| check           |	❌  | ❌ |

</TabItem>
</Tabs>


## FAQs

### どのモデルにコントラクトが必要ですか？

上記の基準を満たすモデルであれば、コントラクトを定義できます。下流で利用される ["public" モデル](model-access) に対してコントラクトを定義することをお勧めします。
- dbt 内部: 他のグループ、他のチーム、および [他の dbt プロジェクト](/best-practices/how-we-mesh/mesh-1-intro) と共有されます。
- dbt 外部: このモデルが予測可能な構造を持つことが期待されるレポート、ダッシュボード、その他のシステムやプロセス。これらの下流での使用は、[exposures](/docs/build/exposures) で反映できます。

### コントラクトとテストの違いは何ですか？

モデルのコントラクトは、返されるデータセットの**形状**を定義します。モデルのロジックまたは入力データがその形状に準拠していない場合、モデルはビルドされません。

[データテスト](/docs/build/data-tests)は、モデルの構築後にその内容を検証するための、より柔軟なメカニズムです。クエリを記述できれば、データテストを実行できます。データテストは、[カスタム重大度しきい値](/reference/resource-configs/severity)など、より柔軟に設定可能です。既に構築済みのモデルに対してクエリを実行したり、[失敗したレコードをデータウェアハウスに保存](/reference/resource-configs/store_failures)できるため、障害を発見した後のデバッグが容易になります。

場合によっては、データテストを同等の制約に置き換えることができます。これには、ビルド時に検証が保証されるという利点があり、データプラットフォームでのコンピューティング（コスト）も削減される可能性があります。データテストを制約に置き換えるための前提条件は次のとおりです。
- データプラットフォームが必要な制約をサポートし、適用できることを確認する。ほとんどのプラットフォームでは `not_null` のみが適用されます。
- モデルを `table` または `incremental` としてマテリアライズする（**`view` や `ephemeral` ではない）
- 各列の `name` と `data_type` を指定して、このモデルの完全なコントラクトを定義する。

**なぜテストは契約の一部ではないのですか？** ソフトウェアAPIの場合と同様に、APIレスポンスの構造が契約です。品質と信頼性（「稼働時間」）もAPIの品質にとって非常に重要な属性ですが、それ自体は契約の一部ではありません。契約が後方互換性のない形で変更された場合、それは破壊的変更となり、メジャーバージョンのアップグレードが必要になります。

### コントラクトのすべての列を定義する必要がありますか？

現在、dbt コントラクトはモデルで定義されているすべての列に適用され、すべての列について明示的な期待値を宣言する必要があります。コントラクトの明示的な宣言は偶然ではなく、この機能の本来の目的です。

同時に、列数の多いモデルの場合、YAML が大量に必要になる可能性があることも理解しています。「推論型」コントラクトのサポートの実現可能性を検討しています。これにより、一部の列に対して制約と厳密なデータ型を定義しながら、本番環境で同じモデルと比較することで、他の列の互換性を破る変更を検出できるようになります。これは「部分的」コントラクトとは異なります。モデル内のすべての列は実行時にチェックされ、YAML コントラクトで明示的に定義されている内容、または比較状態によって暗黙的に定義されている内容と照合されるからです。 「推論された」契約に興味がある場合は、[dbt-core#7432](https://github.com/dbt-labs/dbt-core/issues/7432) に賛成またはコメントしてください。


### 破壊的変更はどのように処理されますか？

以前のプロジェクト状態と比較する際、dbt は下流のコンシューマーに影響を与える可能性のある破壊的変更を探します。破壊的変更が検出された場合、dbt はコントラクトエラーを表示します。

import BreakingChanges from '/snippets/_versions-contracts.md';

<BreakingChanges 
value="Removing a contracted model by deleting, renaming, or disabling it (dbt v1.9 or higher)."
value2="versioned models will raise an error. unversioned models will raise a warning."
/>

詳細については、[コントラクト リファレンス](/reference/resource-configs/contract#detecting-breaking-changes) をご覧ください。
