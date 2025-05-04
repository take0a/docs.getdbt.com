---
title: "メトリックフィルターを使用したディメンションとしてのメトリック"
id: "ref-metrics-in-filters"
description: "メトリックをディメンションとしてメトリック フィルターに追加して、より複雑なメトリックを作成し、より多くの分析情報を得ることができます。"
sidebar_label: "Metrics as dimensions"
---

[メトリクス](/docs/build/metrics-overview)は、アクティブユーザー数や全体的なパフォーマンス傾向など、ビジネス上の意思決定に役立つデータに関する貴重な分析情報をユーザーに提供します。一方、[ディメンション](/docs/build/dimensions)は、ユーザータイプや顧客の注文数などの属性に基づいてデータを分類するのに役立ちます。

情報に基づいたビジネス上の意思決定を行うには、一部の指標では、指標の定義の一部として別の指標の値が必要になります。そこで「ディメンションとしての指標」という考え方が登場します。

このドキュメントでは、指標フィルターを用いて指標をディメンションとして使用し、より複雑な指標を作成し、より多くのインサイトを得る方法について説明します。dbt Cloud バージョン 1.8 以降でご利用いただけます。

## フィルター内でのメトリックの参照

`Metric()` オブジェクト構文を使用して、`where` フィルター内で別のメトリックを参照します。メトリックを参照する関数は、メトリック名とエンティティを1つだけ受け入れます。

```yaml
{{ Metric('metric_name', group_by=['entity_name']) }}
```

### 使用例

例えば、SaaS（Software as a Service）企業が、アクティブ化されたアカウントをカウントしたいとします。この場合、アクティブ化されたアカウントとは、データモデルの実行回数が5回を超えるアカウントと定義されます。

この指標をSQLで表現するには、次の操作を行います。
- アカウントごとのデータモデルの実行回数を計算するクエリを作成します。
- 次に、データモデルの実行回数が5回を超えるアカウントの数をカウントします。

<File name="models/model_name.sql">

```sql
with data_models_per_user as (
    select
        account_id as account,
        count(model_runs) as data_model_runs
    from 
        {{ ref('fct_model_runs') }}
    group by 
        account_id
),

activated_accounts as (
    select
        count(distinct account_id) as activated_accounts
    from 
        {{ ref('dim_accounts') }}
    left join 
        data_models_per_user 
    on 
        {{ ref('dim_accounts') }}.account_id = data_models_per_user.account
    where 
        data_models_per_user.data_model_runs > 5
)

select
    *
from 
    activated_accounts
```
</File>

このSQLクエリは、ユーザーエンティティのディメンションとして「data_model_runs」メトリックを使用し、「activated_accounts」の数を計算します。アカウントエンティティにスコープ設定されたメトリック値に基づいてフィルタリングします。このロジックは、クエリレベルまたはメトリックのYAML設定で記述できます。

#### YAML 構成

[使用例](#usage-example) で説明したのと同じ `activated_accounts` の例を使用して、次の YAML 例では、企業が [セマンティックモデル](/docs/build/semantic-models) と [メトリクス](/docs/build/metrics-overview) を作成し、`Metric()` オブジェクトを使用して `activated_accounts` メトリクスフィルターで `data_model_runs` メトリクスを参照する方法を説明します。

- 2 つのセマンティックモデル (`model_runs` と `accounts`) を作成します。
- データモデルの実行回数をカウントする `measure` と `metric`、およびユーザー数をカウントする別のメジャーを作成します。
- `model_runs` セマンティックモデルで、外部エンティティ `account` を指定します。
- 次に、データモデルの実行回数が 5 回を超えるアカウントをフィルタリングして、`Activated Accounts` メトリクスを作成します。

  <File name="models/metrics/semantic_model.yml">

  ```yaml
  semantic_models:
    - name: model_runs
      ... # Placeholder for other configurations
      entities:
        - name: model_run
          type: primary
        - name: account
          type: foreign
      measures:
        - name: data_model_runs
          agg: sum
          expr: 1
          create_metric: true # The 'create_metric: true' attribute automatically creates the 'data_model_runs' metric.

    - name: accounts
      ... # Placeholder for other configurations
      entities:
        - name: account
          type: primary
      measures:
        - name: accounts
          agg: sum
          expr: 1
          create_metric: true
  metrics:
    - name: activated_accounts
      label: Activated Accounts
      type: simple
      type_params:
        measure: accounts
      filter: |
        {{ Metric('data_model_runs', group_by=['account']) }} > 5
  ```
  </File>

  コマンドライン インターフェースから `dbt sl query --metrics activated_accounts` を実行したときに、システムがメトリック定義に基づいて生成する SQL を詳しく見てみましょう:

- フィルター `{{ Metric('data_model_runs', group_by=['account']) }}` は、前に示した `data_models_per_user` サブクエリに似た SQL を生成します:

	```sql
	select
		sum(1) as data_model_runs,
		account
	from 
		data_model_runs
	group by
		account
	```

- MetricFlow は、このクエリを、group by 要素の `accounts` メジャーによって生成されたクエリに結合し、フィルター条件を適用します:

	```sql
	select
      sum(1) as activated_accounts
	from accounts
  left join (
      select
          sum(1) as data_model_runs, 
		      account
	    from data_model_runs
	    group by 
		      account
  ) as subq on accounts.account = subq.account
  where data_model_runs > 5
	```

  この指標を作成するために使用される中間テーブルは次のとおりです: `data_model_runs` ディメンションを持つアカウント

  | account | data_model runs |
  | --- | --- |
  | 1 | 4 |
  | 2 | 7 |
  | 3 | 9 |
  | 4 | 1 |

  次に、MetricFlow はこのテーブルを、データ モデル実行回数が 5 回を超えるアカウントにフィルターし、次の条件を満たすアカウントの数をカウントします:

  | activated_accounts |
  | --- |
  | 2 |

#### クエリフィルター

クエリレベルのフィルターでメトリクスを使用することもできます。コマンドラインインターフェース (CLI) で次のコマンドを実行すると、前述のSQLクエリと同じものが生成されます。

```dbt sl query --metrics accounts --where "{{ Metric('data_model_runs', group_by=['account']) }} > 5"```

結果の SQL とデータは、`activated_accounts` ではなく `accounts` メトリック名を使用することを除いて同じになります。

## 考慮事項

- メトリックフィルターを使用する場合は、サブクエリが外部クエリに結合できることを確認してください。結合の結果がファンアウト（行数が予期せず増加）しないようにする必要があります。
  - `{{ Metric('data_model_runs', group_by=['account']) }}` を使用してアカウントメジャーをフィルタリングする例は、モデル実行をアカウントレベルで集計するため有効です。
  - ただし、`{{ Metric('data_model_runs', group_by=['model']) }}` を使用して「アカウント」メジャーをフィルタリングすることは、アカウントとモデル実行の間に1対多の関係があるため有効ではなく、データが重複します。
- メトリックは1つのエンティティでのみグループ化できます。複数のエンティティとディメンションによるグループ化のサポートは保留中です。
- 今後、以下のユースケース例の一部において、指標をディメンションとして使用できるようになります。
  - ユーザーセグメント：過去7日間のユーザーによる注文数をディメンションとして使用して、ユーザーをセグメント化します。
  - チャーン予測：アカウントが最初の30日間に送信したサポートチケットの数を使用して、潜在的なチャーンを予測します。
  - アクティベーショントラッキング：サインアップ後、一定日数以内に行われた特定のアクションに基づいて、アカウントまたはユーザーのアクティベーションを定義します。
  - マルチホップ結合を必要とする指標フィルターのサポートは保留中です。
