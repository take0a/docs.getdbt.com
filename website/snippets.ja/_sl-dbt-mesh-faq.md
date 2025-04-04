[dbt メッシュ](/best-practices/how-we-mesh/mesh-1-intro)設定で dbt セマンティック レイヤーを使用する場合は、次のことをお勧めします。

- セマンティック モデルとメトリックを含むスタンドアロン プロジェクトが 1 つあります。
- 次に、セマンティック レイヤーを構築するときに、[2 つの引数を持つ `ref` 関数](/reference/dbt-jinja-functions/ref#ref-project-specific-models)( `ref('project_name', 'model_name')`) を使用して、さまざまなプロジェクトまたはパッケージ間で [dbt モデルを相互参照](/docs/collaborate/govern/project-dependencies) し、セマンティック モデルを作成できます。
- dbt セマンティック レイヤー プロジェクトは、残りのプロジェクト全体で信頼できるグローバル ソースとして機能します。

#### 使用例 
たとえば、`jaffle_finance` プロジェクトに存在するパブリック モデル (`fct_orders`) があるとします。セマンティック モデルを構築するときは、次の構文を使用してモデルを参照します:

<File name="models/metrics/semantic_model_name.yml">

```yaml
semantic_models:
  - name: customer_orders
    defaults:
      agg_time_dimension: first_ordered_at
    description: |
      Customer grain mart that aggregates customer orders.
    model: ref('jaffle_finance', 'fct_orders') # ref('project_name', 'model_name')
    entities:
      ...rest of configuration...
    dimensions:
      ...rest of configuration...
    measures:
      ...rest of configuration...
```
</File>

`model` パラメータでは、`jaffle_finance` プロジェクトで定義されたパブリック モデル `fct_orders` を参照するために、2 つの引数を持つ `ref` 関数を使用していることに注意してください。
<br />
