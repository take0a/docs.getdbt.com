---
title: "Model access"
id: model-access
sidebar_label: "Model access"
description: "Define model access with group capabilities"
---

:::info 「モデルアクセス」は「ユーザーアクセス」とは異なります。

**モデルグループとアクセス** と **ユーザーグループとアクセス** はそれぞれ異なる意味を持ちます。「ユーザーグループとアクセス」は、dbt Cloud で権限を管理するために使用される特定の用語です。詳細については、[ユーザーアクセス](/docs/cloud/manage-access/about-user-access) を参照してください。

今年、マルチプロジェクトのコラボレーションワークフローを開発するにあたり、これら 2 つの概念は密接に関連しています。
- dbt プロジェクトで開発権限を持つユーザーは、そのプロジェクト内の**すべての**モデル（プライベートモデルを含む）を表示および変更できます。
- 同じ dbt Cloud アカウント内で、プロジェクトで開発権限を持たないユーザーは、そのプロジェクトのプライベートモデルを表示できず、パブリックモデルにのみ依存関係を作成できます。
:::

## 関連ドキュメント
* [`groups`](/docs/build/groups)
* [`access`](/reference/resource-configs/access)

## グループ

モデルは、共通の所有者を持つ共通の名称でグループ化できます。例えば、特定のチームが所有するすべてのモデル、または特定のデータソース (`github`) のモデリングに関連するすべてのモデルをグループ化できます。

モデルをグループとして定義する理由は2つあります。
- 暗黙的な関係を、明確な所有者を持つ明示的なグループ化に変換します。グループ間のインターフェース境界を考慮することで、よりクリーンな (より絡み合いの少ない) DAG を構築できます。将来的には、これらのインターフェース境界は、個別のプロジェクト間のインターフェースとして適切になる可能性があります。
- 特定のモデルを「プライベート」アクセスとして指定し、そのグループ内でのみ使用できるようにします。他のモデルは、これらのモデルを参照 (依存関係を取得) できなくなります。将来的には、これらのモデルはプロジェクトに依存する他のチームには表示されなくなり、「パブリック」モデルのみが表示されます。

[dbt プロジェクトの構造化に関するベストプラクティス](/best-practices/how-we-structure/1-guide-overview)に従っている場合は、おそらく既にサブディレクトリを使用して dbt プロジェクトを整理しているでしょう。`group` ラベルをサブディレクトリ全体に一括で適用するのは簡単です:

<File name="dbt_project.yml">

```yml
models:
  my_project_name:
    marts:
      customers:
        +group: customer_success
      finance:
        +group: finance
```

</File>

各モデルは1つの `group` にのみ所属でき、グループはネストできません。モデルのYAMLまたはファイル内設定で異なる `group` を設定した場合、プロジェクトレベルで適用された `group` が上書きされます。


import ModelGovernanceRollback from '/snippets/_model-governance-rollback.md';

<ModelGovernanceRollback />

## アクセス修飾子

一部のモデルは実装の詳細であり、関連するモデルのグループ内でのみ参照されることを目的としています。その他のモデルは、[ref](/reference/dbt-jinja-functions/ref) 関数を通じてグループやプロジェクト間でアクセスできるようにする必要があります。モデルには、[アクセス修飾子](https://en.wikipedia.org/wiki/Access_modifiers) を設定することで、意図するアクセスレベルを示すことができます。

| Access    | Referenceable by                       |
|-----------|----------------------------------------|
| private   | 同じグループ                             |
| protected | 同じプロジェクト（またはパッケージとしてインストール） |
| public    | 任意のグループ、パッケージ、またはプロジェクト。定義したら、変更を適用するために本番ジョブを再実行します。 |

サポートされているアクセス範囲外でモデルを参照しようとすると、エラーが表示されます:

```shell
dbt run -s marketing_model
...
dbt.exceptions.DbtReferenceError: Parsing Error
  Node model.jaffle_shop.marketing_model attempted to reference node model.jaffle_shop.finance_model, 
  which is not allowed because the referenced node is private to the finance group.
```

デフォルトでは、すべてのモデルは `protected` されています。つまり、同じプロジェクト内の他のモデルは、グループに関係なく、それらのモデルを参照できます。これは主に、既存のモデルセットにグループを割り当てる際の下位互換性を確保するためです。グループ割り当て間で既に参照が存在する可能性があるためです。

ただし、グループ間での共有を意図して設計されていないモデルに、他のプロジェクトリソースが依存するのを防ぐため、新しいモデルのアクセス修飾子を `private` に設定することをお勧めします。

<File name="models/marts/customers.yml">

```yaml
# First, define the group and owner
groups:
  - name: customer_success
    owner:
      name: Customer Success Team
      email: cx@jaffle.shop

# Then, add 'group' + 'access' modifier to specific models
models:
  # This is a public model -- it's a stable & mature interface for other teams/projects
  - name: dim_customers
    group: customer_success
    access: public
    
  # This is a private model -- it's an intermediate transformation intended for use in this context *only*
  - name: int_customer_history_rollup
    group: customer_success
    access: private
    
  # This is a protected model -- it might be useful elsewhere in *this* project,
  # but it shouldn't be exposed elsewhere
  - name: stg_customer__survey_results
    group: customer_success
    access: protected
```

</File>

`materialized` が `ephemeral` に設定されているモデルでは、アクセスプロパティを public に設定できません。

例えば、モデル設定が次のように設定されているとします:

<File name="models/my_model.sql">

```sql

{{ config(materialized='ephemeral') }}

```

</File>

そして、モデル アクセスが定義されます:

<File name="models/my_project.yml">

```yaml

models:
  - name: my_model
    access: public

```

</File>

次のエラーが発生します:

```
❯ dbt parse
02:19:30  Encountered an error:
Parsing Error
  Node model.jaffle_shop.my_model with 'ephemeral' materialization has an invalid value (public) for the access field
```

## FAQs

### モデルアクセスとデータベース権限の関係は？

これらは異なります！

モデルに `access: public` を指定しても、dbt がモデルをマテリアライズする際に、データプラットフォーム内のすべてのユーザーまたはロールにそのモデルの `select` 権限を自動的に付与するわけではありません。すべてのモデル/スキーマに対するデータベース権限の管理は、お客様と組織の状況に応じて完全に制御できます。

もちろん、dbt は [`grants` 設定](/reference/resource-configs/grants) やその他の柔軟なメカニズムを用いて、これを容易に実現できます。例えば、以下のようになります。
- パブリックモデルへの下流クエリ実行者へのアクセスを許可する
- プライベートモデルへのアクセスを制限する（デフォルト/将来のアクセス権限を取り消す、または別のスキーマに配置する）

マルチプロジェクトコラボレーションの開発を進めていく中で、`access: public` は他のチームがそのモデルへの依存関係を取得できるようになることを意味します。これは、他のチームが基盤となるデータセットからの選択アクセスをリクエストし、お客様がそのアクセスを許可していることを前提としています。

### 別のプロジェクトからモデルを参照するにはどうすればよいですか？

別のプロジェクトからモデルを参照するには、次の 2 つの方法があります。
1. [プロジェクト依存関係](/docs/collaborate/govern/project-dependencies): dbt Cloud Enterprise では、プロジェクト依存関係を使用してモデルを参照できます。dbt Cloud は、バックグラウンドでメタデータ サービスを使用して参照を解決するため、チーム間や大規模なコラボレーションを効率的に実現できます。
2. ["パッケージ" 依存関係](/docs/build/packages): 別のプロジェクトからモデルを参照するもう 1 つの方法は、別のプロジェクトをパッケージ依存関係として扱うことです。この場合、別のプロジェクトをパッケージとしてインストールする必要があります。これには、そのプロジェクトの完全なソースコードと上流の依存関係も含まれます。

### パッケージで定義されたモデルへのアクセスを制限するにはどうすればよいですか？

パッケージからインストールされたソースコードは、ランタイム環境の一部となります。マクロを呼び出したり、モデルを実行したりすることは、自分のプロジェクトで定義したマクロやモデルであるかのように行うことができます。

そのため、パッケージで定義されたモデルに対するモデルアクセス制限は、デフォルトで「オフ」になっています。そのパッケージのモデルは、`access` 修飾子に関わらず参照できます。

パッケージとしてインストールされたプロジェクトでは、外部からの `ref` アクセスを、そのパッケージに含まれる公開モデルのみに制限することもできます。パッケージのメンテナーは、`dbt_project.yml` で `restrict-access` 設定を `True` に設定することでこれを実現します。

デフォルトでは、この設定の値は `False` です。これは以下のことを意味します。
- パッケージ内の `access: protected` が指定されたモデルは、ルートプロジェクト内のモデルから、同じプロジェクトで定義されているかのように参照できます。
- パッケージ内の `access: private` が指定されたモデルは、ルートプロジェクト内のモデルから参照できます。ただし、それらのモデルが同じ `group` 設定を持っている必要があります。

`restrict-access: True` の場合：
- パッケージ外からそのパッケージ内の protected または private なモデルへの `ref` は失敗します。
- パッケージ外から参照できるのは、 `access: public` が指定されたモデルのみです。

<File name="dbt_project.yml">

```yml
restrict-access: True  # default is False
```

</File>

