---
title: "Defer"
---

Deferは、上流の親を事前にビルドすることなく、モデルやテストのサブセットを[サンドボックス環境](/docs/environments-in-dbt)で実行できるようにする強力な機能です。これにより、大規模プロジェクトで少数のモデルをテストする場合に、時間と計算リソースを節約できます。

<Lightbox src src="/img/docs/reference/defer-diagram.png" width="50%" title="Use 'defer' to modify end-of-pipeline models by pointing to production models, instead of running everything upstream." />

defer を使用するには、以前の dbt 呼び出しのマニフェストを `--state` フラグまたは環境変数に渡す必要があります。`state:` 選択方法と組み合わせることで、これらの機能により「スリム CI」が実現します。[state](/reference/node-selection/syntax#about-node-selection) の詳細については、こちらをご覧ください。

異なるユースケースで同様の機能を実現する代替コマンドとして `dbt clone` があります。詳細については、[clone](/reference/commands/clone#when-to-use-dbt-clone-instead-of-deferral) のドキュメントをご覧ください。

`--state`/`DBT_STATE` と `--defer-state`/`DBT_DEFER_STATE` にそれぞれ異なるマニフェストへのパスを渡すことで、`state:modified` と `--defer` で別々の状態を使用できます。これにより、ある環境または過去の時点の論理状態と比較し、別の環境または時点の適用済み状態に遅延させるといった、よりきめ細かな制御が可能になります。`--defer-state` が指定されていない場合、遅延は `--state` に指定されたマニフェストを使用します。ほとんどの場合、論理的な変更を本番環境と比較し、未構築の上流リソースについては本番環境に「フェイルオーバー」するなど、両方で同じ状態を使用することになります。

### 使用法

```shell
dbt run --select [...] --defer --state path/to/artifacts
dbt test --select [...] --defer --state path/to/artifacts
```

デフォルトでは、dbt は [`target`](/reference/dbt-jinja-functions/target) 名前空間を使用して `ref` 呼び出しを解決します。

`--defer` が有効になっている場合、dbt は ref 呼び出しを状態マニフェストを使用して解決しますが、次の条件を満たす場合のみです。

1. ノードが選択されたノードに含まれていない。_かつ_
2. ノードがデータベースに存在しない（または `--favor-state` が使用されている）。

エフェメラルモデルは、他の `ref` 呼び出しの「パススルー」として機能するため、遅延されることはありません。

defer を使用する場合、本番環境データセット、開発データセット、またはその両方から選択できます。ただし、予期しない結果が生じる可能性があるので注意してください。
- 開発環境では環境固有の制限を適用し、本番環境では適用しない場合、予想よりも多くのデータが選択される可能性があります。
- 複数の親（例：リレーションシップ）に依存するテストを実行する場合（複数の環境をまたいでテストするため）

遅延を実行するには、フラグを明示的に渡すか、環境変数（DBT_DEFER と DBT_STATE）を設定することで、`--defer` と `--state` の両方を設定する必要があります。dbt Cloud を使用する場合は、[CI ジョブの設定方法](/docs/deploy/continuous-integration) をご覧ください。


#### 状態を優先

`--favor-state` が指定された場合、dbt は `--state directory` 内のノード定義を優先します。ただし、指定されたノードが選択されたノードの一部でもある場合は、この設定は適用されません。

### 例

ローカル開発環境では、すべてのモデルをターゲットスキーマ「dev_alice」内に作成します。本番環境では、同じモデルが「prod」というスキーマ内に作成されます。

本番環境で実行した dbt によって生成された [アーティファクト](/docs/deploy/artifacts) (つまり「manifest.json」) にアクセスし、「prod-run-artifacts」というローカルディレクトリにコピーします。

### 実行

`model_b` に取り組んでいました。

<File name='models/model_b.sql'>

```sql
select

    id,
    count(*)

from {{ ref('model_a') }}
group by 1
```

変更内容をテストしたいのですが、開発スキーマ `dev_alice` には何も存在しません。

</File>

<Tabs
  defaultValue="no_defer"
  values={[
    { label: 'Standard run', value: 'no_defer', },
    { label: 'Deferred run', value: 'yes_defer', },
  ]
}>

<TabItem value="no_defer">

```shell
dbt run --select "model_b"
```

<File name='target/run/my_project/model_b.sql'>

```sql
create or replace view dev_me.model_b as (

    select

        id,
        count(*)

    from dev_alice.model_a
    group by 1

)
```

以前にこの開発環境で `model_a` を実行していなければ、 `dev_alice.model_a` は存在せず、データベース エラーが発生します。

</File>
</TabItem>

<TabItem value="yes_defer">

```shell
dbt run --select "model_b" --defer --state prod-run-artifacts
```

<File name='target/run/my_project/model_b.sql'>

```sql
create or replace view dev_me.model_b as (

    select

        id,
        count(*)

    from prod.model_a
    group by 1

)
```

</File>

`model_a` が選択されていないため、dbt は `dev_alice.model_a` が存在するかどうかを確認します。存在しない場合、dbt は `{{ ref('model_a') }}` のすべてのインスタンスを `prod.model_a` に解決します。

</TabItem>
</Tabs>

### テスト

`model_a` と `model_b` 間の参照整合性を確立する `relationships` テストもあります:

<File name='models/resources.yml'>

```yml
version: 2

models:
  - name: model_b
    columns:
      - name: id
        tests:
          - relationships:
              to: ref('model_a')
              field: id
```

(`model_b` のすべてのデータは `model_a` から取得する必要があったため、少しばかげていますが、信じられないかもしれません。)

</File>

<Tabs
  defaultValue="no_defer"
  values={[
    { label: 'Without defer', value: 'no_defer', },
    { label: 'With defer', value: 'yes_defer', },
  ]
}>

<TabItem value="no_defer">

```shell
dbt test --select "model_b"
```

<File name='target/compiled/.../relationships_model_b_id__id__ref_model_a_.sql'>

```sql
select count(*) as validation_errors
from (
    select id as id from dev_alice.model_b
) as child
left join (
    select id as id from dev_alice.model_a
) as parent on parent.id = child.id
where child.id is not null
  and parent.id is null
```

`relationships` テストには `model_a` と `model_b` の両方が必要です。前回の `dbt run` で `model_a` をビルドしなかったため、`dev_alice.model_a` は存在せず、このテストクエリは失敗します。

</File>
</TabItem>

<TabItem value="yes_defer">

```shell
dbt test --select "model_b" --defer --state prod-run-artifacts
```

<File name='target/compiled/.../relationships_model_b_id__id__ref_model_a_.sql'>

```sql
select count(*) as validation_errors
from (
    select id as id from dev_alice.model_b
) as child
left join (
    select id as id from prod.model_a
) as parent on parent.id = child.id
where child.id is not null
  and parent.id is null
```

</File>

dbtは`dev_alice.model_a`が存在するかどうかを確認します。存在しない場合、dbtはスキーマテスト内のものも含め、`{{ ref('model_a') }}`のすべてのインスタンスを解決し、代わりに`prod.model_a`を使用します。クエリは成功します。環境間で参照整合性をテストする必要があるかどうかは別の問題です。

</TabItem>
</Tabs>

## 関連ドキュメント

- [dbt Cloud での defer の使用](/docs/cloud/about-cloud-develop-defer)
- [on_configuration_change](/reference/resource-configs/on_configuration_change)

