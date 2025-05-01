---
title: 結合
id: join-logic
description: "結合を使用すると、異なるテーブルのデータを組み合わせて新しいメトリックを作成できます。"
sidebar_label: "Joins"
tags: [Metrics, Semantic Layer]
---

結合はMetricFlowの強力な機能であり、異なるセマンティックモデルで定義されている場所に関係なく、クエリ時にすべての有効なディメンションをメトリクスで使用できるようにするプロセスを簡素化します。
結合を使用すると、異なるセマンティックモデルのメジャーを使用してメトリクスを作成することもできます。

結合では、セマンティックモデル構成で定義された「エンティティ」をテーブル間の結合キーとして使用します。
セマンティックモデルでエンティティが定義されていると仮定すると、MetricFlowはセマンティックモデルをノード、結合パスをエッジとして使用してグラフを作成し、自動的に結合を実行します。
MetricFlowは、エンティティタイプに基づいて適切な結合タイプを選択し、他のテーブルとのファンアウト結合やキャズム結合を回避します。

<Expandable alt_header="ファンアウト結合またはキャズム結合とは何ですか?" >
- ファンアウト結合とは、あるテーブルの 1 つの行が別のテーブルの複数の行に結合され、入力行よりも出力行が多くなる結合です。
- キャズム結合とは、2 つのテーブルが中間テーブルを介して多対多の関係にあり、結合の結果データが重複または欠落する場合です。
</Expandable>

## 結合の種類

:::tip Joins are auto-generated
MetricFlow は、定義されたセマンティックオブジェクトに必要な結合を自動的に生成するため、新しいセマンティックモデルや設定ファイルを作成する必要はありません。

このセクションでは、エンティティで使用できるさまざまな種類の結合と、それらをクエリする方法について説明します。
:::

Metricflow は、以下の特定の結合戦略を使用します。

- `fct` モデルと `dim` モデルを結合する際には、主に左結合を使用します。左結合では、「ベース」テーブルのすべての行が保持され、結合先のテーブルから一致する行が含まれます。
- 複数の `fct` モデルを含むクエリの場合、MetricFlow は完全外部結合を使用して、特定のテーブルで一部の `dim` モデルまたは `fct` モデルが欠落している場合でも、すべてのデータポイントが確実に取得されるようにします。
- MetricFlow は、ファンアウト結合とキャズム結合の使用を制限します。

MetricFlow が実際に結合を処理する方法の詳細については、[SQL の例](#sql-examples) を参照してください。

次の表は、リスクの高い結合の作成を防ぐために、特定のエンティティタイプに基づいて許可される結合を示しています。
この表は、特に指定がない限り、主に左結合を表しています。
複数の `fct` モデルが関係するシナリオでは、MetricFlow は完全外部結合を使用します。

| entity type - Table A | entity type - Table B | Join type            |
|---------------------------|---------------------------|----------------------|
| Primary                   | Primary                   | ✅ Left                 |
| Primary                   | Unique                    | ✅ Left                 |
| Primary                   | Foreign                   | ❌ Fan-out (Not allowed) |
| Unique                    | Primary                   | ✅ Left                 |
| Unique                    | Unique                    | ✅ Left                 |
| Unique                    | Foreign                   | ❌ Fan-out (Not allowed) |
| Foreign                   | Primary                   | ✅ Left                 |
| Foreign                   | Unique                    | ✅ Left                 |
| Foreign                   | Foreign                   | ❌ Fan-out (Not allowed) |

### セマンティック検証

MetricFlow は、データプラットフォームで `explain` クエリを実行することでセマンティック検証を行い、生成された SQL がエラーなく実行されることを確認します。この検証には以下の内容が含まれます。

- 参照されているすべてのテーブルと列が存在することの確認。
- データプラットフォームが `date_diff(x, y)` などの SQL 関数をサポートしていることの確認。
- マルチホップ結合における曖昧な結合やパスのチェック。

検証に失敗した場合、MetricFlow はクエリ実行前にユーザーが対処すべきエラーを表示します。

## 例

次の例では、共通のエンティティを持つ2つのセマンティックモデルを使用し、2つのセマンティックモデル間の結合を必要とするMetricFlowクエリを示します。
2つのセマンティックモデルは次のとおりです。
- `transactions`
- `user_signup`

```yaml
semantic_models:
  - name: transactions
    entities:
      - name: id
        type: primary
      - name: user
        type: foreign
        expr: user_id
    measures:
      - name: average_purchase_price
        agg: avg
        expr: purchase_price
  - name: user_signup
    entities:
      - name: user
        type: primary
        expr: user_id
    dimensions:
      - name: type
        type: categorical
```

- MetricFlow は、`user_id` を結合キーとして使用し、2 つのセマンティックモデル `transactions` と `user_signup` をリンクします。これにより、`transactions` セマンティックモデルの `average_purchase_price` メトリクスを、`user_signup` セマンティックモデルの `type` ディメンションでグループ化された状態でクエリできます。
- `average_purchase_price` メジャーは `transactions` で定義されており、`user_id` は外部エンティティです。一方、`user_signup` は `user_id` をプライマリエンティティとして持っています。
- `user_id` は `transactions` では外部キーであり、`user_signup` ではプライマリキーであるため、MetricFlow は `transactions` と `user_signup` を結合する左結合を実行し、`transactions` で定義された `average_purchase_price` メジャーにアクセスします。
- 異なるセマンティックモデルからディメンションをクエリするには、編集ツールでエンティティを結合した後、ディメンション名に二重アンダースコア（またはdunder）を追加します。次のクエリでは、「user_id__type」が「--group-by」フラグを使用してディメンションとして追加されています（「type」がディメンションです）。

```yaml 
dbt sl query --metrics average_purchase_price --group-by metric_time,user_id__type # In dbt Cloud
```

```yaml 
mf query --metrics average_purchase_price --group-by metric_time,user_id__type # In dbt Core
```

#### SQL の例

以下の SQL の例は、MetricFlow が左結合と完全外部結合の両方のシナリオを実際にどのように処理するかを示しています。

<Tabs>
<TabItem value="左結合のSQL例"> 

前の例の `transactions` および `user_signup` セマンティック モデルを使用すると、これら 2 つのセマンティック モデル間の左結合が示されます。

```sql
select
  transactions.user_id,
  transactions.purchase_price,
  user_signup.type
from transactions
left outer join user_signup
  on transactions.user_id = user_signup.user_id
where transactions.purchase_price is not null
group by
  transactions.user_id,
  user_signup.type;
```
</TabItem>

<TabItem value="外部結合のSQL例"> 

複数の「fct」モデル（例えば「sales」と「returns」）がある場合、MetricFlowは完全外部結合を使用してすべてのデータポイントを確実に取得します。

この例は、「sales」セマンティックモデルと「returns」セマンティックモデル間の完全外部結合を示しています。

```sql
select
  sales.user_id,
  sales.total_sales,
  returns.total_returns
from sales
full outer join returns
  on sales.user_id = returns.user_id
where sales.user_id is not null or returns.user_id is not null;
```

</TabItem>
</Tabs>

## マルチホップ結合

MetricFlow では、エンティティのグラフ内のあるテーブルから別のテーブルに移動することで、グラフ全体にわたってメジャーとディメンションを結合できます。
これは「マルチホップ結合」と呼ばれます。

MetricFlow は最大 3 つのテーブルを結合でき、最大 2 ホップまでのマルチホップ結合をサポートしています。これにより、以下のことが可能になります。
- 曖昧なパスのない複雑なデータ分析が可能になります。
- 「注文」テーブルから「顧客」テーブル、そして「国」テーブルへと移動するなど、データモデル内のナビゲーションをサポートします。

同じデータへの複数のルートによる混乱を防ぐため、直接の3ホップパスは制限されていますが、MetricFlowでは、結合が2ホップを超えずにディメンションに到達できる場合、3つ以上のテーブルを結合できます。

例えば、「country」と「region」という2つのモデルがあり、顧客が国にリンクされ、国が地域にリンクされている場合、これらすべてを1つのSQLクエリで結合し、「customer__country_country_name」で「orders」を分析できますが、「customer__country__region_name」では分析できません。

![Multi-Hop-Join](/img/docs/building-a-dbt-project/multihop-diagram.png "Example schema for reference")

スキーマを次の 3 つの MetricFlow セマンティック モデルに変換し、売上テーブルの `purchase_price` メジャーと `country_dim` テーブルの `country_name` ディメンションを使用してメトリック「国別の平均購入価格」を作成する方法に注目してください。

```yaml
semantic_models:
  - name: sales
    defaults:
      agg_time_dimension: first_ordered_at
    entities:
      - name: id
        type: primary
      - name: user_id
        type: foreign
    measures:
      - name: average_purchase_price
        agg: avg
        expr: purchase_price
    dimensions:
      - name: metric_time
        type: time
        type_params:
  - name: user_signup
    entities:
      - name: user_id
        type: primary
      - name: country_id
        type: unique
    dimensions:
      - name: signup_date
        type: time
      - name: country_dim

  - name: country
    entities:
      - name: country_id
        type: primary
    dimensions:
      - name: country_name
        type: categorical
```

### マルチホップ結合のクエリ

マルチホップ結合を使用せずにディメンションをクエリするには、`entity__dimension` のように、エンティティダブルアンダースコア（dunder）ディメンション構文で完全修飾ディメンション名を使用できます。

マルチホップ結合によって取得されるディメンションの場合は、`user_id` のように、エンティティパスをリストとして追加で指定する必要があります。

