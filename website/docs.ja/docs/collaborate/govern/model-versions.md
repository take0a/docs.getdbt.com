---
title: "Model versions"
id: model-versions
sidebar_label: "Model versions"
description: ライフサイクル管理を支援するバージョンモデル"
keyword: governance, model version, model versioning, dbt model versioning
---

<VersionBlock lastVersion="1.8">

:::info New functionality
This functionality is new in v1.5 — if you have thoughts, participate in [the discussion on GitHub](https://github.com/dbt-labs/dbt-core/discussions/6736)!
:::

</VersionBlock>

import VersionsCallout from '/snippets/_model-version-callout.md';

<VersionsCallout />

API のバージョン管理は、ソフトウェアエンジニアリングにおいて難しい問題です。根本的な問題は、API の制作者と利用者の間に相反するインセンティブがあることです。
- API の制作者は、API のロジックと構造を変更できる必要があります。レガシーエンドポイントを永続的に維持するにはコストがかかりますが、下流のユーザーの信頼を失うことの方がはるかに大きなコストがかかります。
- API の利用者は、API の安定性を信頼する必要があります。つまり、クエリが継続的に機能し、警告なしに中断されることはありません。新しい API バージョンへの移行にはコストがかかりますが、計画外の移行ははるかに大きなコストがかかります。

最終的な dbt モデルを他のチームやシステムと共有する場合、そのモデルは API のように動作します。そのモデルの制作者が大幅な変更を加える必要が生じた場合、下流のユーザーのクエリが中断されるのをどのように回避できるでしょうか？

モデルのバージョン管理は、この問題に思慮深く、真正面から取り組むためのツールです。目標は、問題を完全に解決することでも、実際よりも簡単または単純であるかのように装うことでもありません。

## 関連ドキュメント
- [`versions`](/reference/resource-properties/versions)
- [`latest_version`](/reference/resource-properties/latest_version)
- [`include` と `exclude`](/reference/resource-properties/versions#include)
- [`version` 引数を指定した `ref`](/reference/dbt-jinja-functions/ref#versioned-ref)

## モデルにバージョン管理が必要な理由

モデルが ["コントラクト"](/docs/collaborate/govern/model-contracts) (構造に関する一連の保証) を定義している場合、以前の保証を破るような方法でモデルの構造を変更することも可能になります。これは、列の削除や名前変更といった明白な変更から、データ型や null 値許容度の変更といったより微妙な変更まで多岐にわたります。

一つのアプローチは、モデルを本番環境にデプロイしたらすぐに、すべてのモデル利用者に互換性を破る変更への対応を強制することです。これは多くの小規模組織や、まだ成熟していないデータモデルセットを迅速に反復処理する場合に、実際には適切な解決策です。しかし、それ以上の規模になると、あまり拡張性がありません。

大規模組織で成熟したモデルをdbt内外のクエリに適用する場合、モデルオーナーは**モデルバージョン**を使用して次のことを行うことができます。
- 「プレリリース」の変更をテストする（本番環境、下流システム）
- 最新バージョンにアップグレードし、信頼できる正規のソースとして使用する
- 「古い」バージョンからの移行期間を提供する

移行期間中、そのモデルが下流で使用されている場所では、特定のバージョンで参照され続ける可能性があります。

dbt Core 1.6 では、[`deprecation_date`](/reference/resource-properties/deprecation_date) を指定することで、**モデルの非推奨化** をファーストクラスでサポートできるようになりました。モデルバージョンと非推奨化を組み合わせることで、モデル作成者は古いモデルを _廃止_ し、モデル利用者は互換性を破る変更を _移行_ する時間を確保できます。これは、組織全体で変更を管理する方法です。新しいバージョンを開発し、最新バージョンに更新し、古いバージョンを非推奨として設定し、下流の参照を更新してから、古いバージョンを削除します。

ここには、下流のコードを頻繁に移行するコストと、データウェアハウスでモデルの複数のバージョンを具体化するコスト（および煩雑さ）という、現実的なトレードオフが存在します。モデルのバージョンによってその問題がなくなるわけではありませんが、廃止日を設定し、消費者が古いバージョンからスムーズに移行するための明確な期間を通知することで、移行コストに既知の境界が設定されます。

## モデルのバージョン管理はいつ行うべきでしょうか?

import ModelGovernanceRollback from '/snippets/_model-governance-rollback.md';

<ModelGovernanceRollback />

モデルの規約を強制することで、dbt は列名やデータ型への意図しない変更を検出し、下流のクエリ実行者に大きな負担をかける可能性があります。これらの変更を意図的に行う場合は、新しいモデルバージョンを作成する必要があります。新しい列の追加や既存の列の計算におけるバグの修正など、互換性に影響のない変更を行う場合は、新しいバージョンは必要ありません。

もちろん、モデルの定義を他の方法で変更することも可能です。たとえば、列の名前、データ型、強制可能な特性を変更せずに列を再計算するなどです。しかし、その結果、下流のクエリ実行者に表示される結果が大幅に変更される可能性があります。

これは常に判断が求められます。広く使用されているモデルのメンテナーであるあなたは、何がバグ修正で何が予期しない動作変更かを最もよく理解しているはずです。

モデルバージョンの廃止と移行のプロセスには、実際の作業が必要であり、チーム間での綿密な調整が必要になる可能性があります。可能な限り、互換性のある変更を選択する必要があります。しかし、これらの互換性のある追加により、必然的に、最も重要なモデルに多くの未使用または非推奨の列が残ってしまいます。

小さな変更ごとに常に新しいバージョンを追加するのではなく、予測可能な頻度（年に1～2回、事前に十分に連絡）でモデルの「最新」バージョンを更新し、使用されなくなった列を削除することをお勧めします。

## これは「バージョン管理」とどう違うのでしょうか？

[バージョン管理](/docs/collaborate/git-version-control)を使用すると、チームは単一のコードリポジトリで同時に共同作業を行い、変更間の競合を管理し、本番環境へのデプロイ前に変更をレビューできます。その意味で、バージョン管理は、dbt プロジェクト全体のデプロイをバージョン管理するための不可欠なツールであり、常に `main` ブランチの最新の状態を維持します。通常、プロジェクトコードは一度に 1 つのバージョンのみが環境にデプロイされます。問題が発生した場合は、コミットまたはプルリクエストを元に戻したり、データプラットフォームの「タイムトラベル」機能を利用したりすることで、変更をロールバックできます。

モデルのソースコード（SQL または Python での論理定義、または関連する構成）を更新すると、dbt は [プロジェクトを以前の状態と比較](/reference/node-selection/syntax#about-node-selection) できるため、変更されたモデルと変更後の下流のモデルのみを再構築できます。この方法により、モデルへの変更を開発し、CI で迅速にテストし、本番環境に効率的にデプロイすることが可能になります。これらはすべてバージョン管理システムを介して調整されます。

**バージョン管理されたモデルは異なります。** モデルの「バージョン」を定義するのは、dbt の内外を問わず、チームの管理下にない人、システム、プロセスがモデルに依存している場合に適しています。すべてを単純に移行することも、気まぐれにクエリを壊すこともできません。明確な差分と廃止予定日を含む移行パスを提供する必要があります。

モデルの複数のバージョンが同じコードリポジトリに同時に存在し、同じデータ環境に同時にデプロイされます。これは、Web API のバージョン管理方法に似ています。複数のバージョンが同時に存在し、2 つまたは 3 つで、それ以上ではありません。時間の経過とともに、新しいバージョンがオンラインになり、古いバージョンは廃止されます。

## これは、単に新しいモデルを作成するのとどう違うのでしょうか？

正直なところ、ほんの少しの違いしかありません！特別なことはほとんどなく、それは仕様です。

これまでも、コピー＆ペーストして新しいモデルファイルを作成し、「dim_customers_v2.sql」という名前を付けることはできました。では、なぜ「本格的な」バージョン管理モデルを選ぶべきなのでしょうか？

バージョン管理モデルの**作成者**として：
- すべてのライブバージョンをコードベース全体に分散させるのではなく、1か所で追跡できます。
- モデルの設定を再利用し、バージョン間の差分だけを強調表示できます。
- モデルが「最新」、「プレリリース」、「古い」バージョンに基づいて、ビルドする（またはしない）モデルを選択できます。
- dbt は、新しいバージョンが利用可能になったとき、または廃止予定になったときに、バージョン管理モデルの利用者に通知します。

バージョン管理されたモデルの**コンシューマー**として:
- 一貫性のある `ref` を使用し、特定のライブバージョンに固定するオプションがあります。
- バージョン管理されたモデルのライフサイクル全体を通して通知されます。

モデルのすべてのバージョンは、モデルの元の名前を保持します。それらは、定義されているファイルの名前ではなく、その名前で `ref` されます。デフォルトでは、`ref` は最新バージョン（そのモデルのメンテナーによって宣言されたバージョン）に解決されますが、`version` キーワードを使用して、モデルの特定のバージョンを `ref` することもできます。

`dim_customers` に 3 つのバージョンが定義されているとします。`v2` は「最新」、`v3` は「プレリリース」、`v1` はまだ廃止予定期間内の古いバージョンです。 `v2` は最新バージョンであるため、特別な扱いを受けます。ファイル内ではサフィックスなしで定義でき、バージョンピンが指定されていない場合は `ref('dim_customers')` は `v2` として解決されます。以下の表は、標準的な規則をまとめたものです。

| v | version    | `ref` syntax                                          | File name                                       | Database relation                                                        |
|---|------------|-------------------------------------------------------|-------------------------------------------------|--------------------------------------------------------------------------|
| 3 | "prerelease" | `ref('dim_customers', v=3)`                           | `dim_customers_v3.sql`                          | `analytics.dim_customers_v3`                                             |
| 2 | "latest"     | `ref('dim_customers', v=2)` **and** `ref('dim_customers')`  | `dim_customers_v2.sql` **or** `dim_customers.sql` | `analytics.dim_customers_v2` **and** `analytics.dim_customers` (recommended) |
| 1 | "old"        |  `ref('dim_customers', v=1)`                           | `dim_customers_v1.sql`                          | `analytics.dim_customers_v1`                                             |

以下の実装セクションで説明するように、バージョン管理されたモデルでは、YAML プロパティと設定の大部分を再利用できます。各バージョンでは、共有属性セットとの違いを記述するだけで済みます。これにより、バージョン管理されたモデルの作成者は、数十または数百の列を持つモデルでは検出が困難なバージョン間の差異を強調表示し、現在稼働中のモデルのすべてのバージョンを 1 か所で明確に追跡できるようになります。

dbt は [`version` ベースの選択](/reference/node-selection/methods#version) もサポートしています。たとえば、[default YAML selector](/reference/node-selection/yaml-selectors#default) を定義して、開発環境では古いモデルバージョンが実行されないようにすることができます。ただし、開発終了や移行期間中は、本番環境で古いモデルバージョンを実行し続けます。(これらのモデルに `tags` を適用し、時間の経過とともにそれらのタグを循環させることで、同様のことを実現できます。)

<File name="selectors.yml">

```yml
selectors:
  - name: exclude_old_versions
    default: "{{ target.name == 'dev' }}"
    definition:
      method: fqn
      value: "*"
      exclude:
        - method: version
          value: old
```

</File>

dbt はこれらのモデルが実際には同じモデルであることを認識しているため、新しいバージョンが利用可能になったときや古いバージョンが廃止される予定になったときに下流の消費者に通知できます。

```bash
Found an unpinned reference to versioned model 'dim_customers'.
Resolving to latest version: my_model.v2
A prerelease version 3 is available. It has not yet been marked 'latest' by its maintainer.
When that happens, this reference will resolve to my_model.v3 instead.

  Try out v3: {{ ref('my_dbt_project', 'my_model', v='3') }}
  Pin to  v2: {{ ref('my_dbt_project', 'my_model', v='2') }}
```

## モデルの新バージョンを作成する方法

多くの場合、まだバージョン管理されていないモデルから始めます。`dim_customers` が、強制的な契約を持つシンプルなスタンドアロンモデルだった頃を振り返ってみましょう。ここでは、単純化のために `customer_id` と `country_name` の2つの列しかないと仮定します。ただし、成熟したモデルの多くは、より多くの列を持つことになります。

<File name="models/dim_customers.sql">

```sql
-- lots of sql

final as (
  
    select
        customer_id,
        country_name
    from ...

)

select * from final
```

</File>

<File name="models/schema.yml">

```yaml
models:
  - name: dim_customers
    config:
      materialized: table
      contract:
        enforced: true
    columns:
      - name: customer_id
        description: This is the primary key
        data_type: int
      - name: country_name
        description: Where this customer lives
        data_type: varchar
```

</File>

モデルに互換性のない変更を加える必要があるとします。つまり、信頼性が低下した `country_name` 列を削除する必要があるとします。まず、これらの互換性のない変更を含む新しいモデルファイル（SQL または Python）を作成します。

デフォルトの規則では、新しいファイル名には `_v<version>` サフィックスを付けます。ここでは、`dim_customers_v2.sql` という新しいファイルを作成します。（既存のモデルファイルはまだ「最新」バージョンなので、名前を変更する必要はありません。）

<File name="models/dim_customers_v2.sql">

```sql
-- lots of sql

final as (
  
    select
        customer_id
        -- country_name has been removed!
    from ...

)

select * from final
```

</File>

これで、`dim_customers_v2` のプロパティと設定を、`dim_customers` とは実質的には何の関係もなく、類似点のみを持つ新しいスタンドアロンモデルとして定義できるようになりました。代わりに、これらが同じモデルのバージョンであり、どちらも `dim_customers` という名前であることを宣言します。これらのモデルに共通のプロパティを定義し、**単に** それらの差分をハイライト表示することができます。（あるいは、各モデルのバージョンを完全な仕様で定義し、共通の値を繰り返すこともできます。）

<Tabs>
<TabItem value="Diffs only (recommended)">

<File name="models/schema.yml">

```yaml
models:
  - name: dim_customers
    latest_version: 1
    config:
      materialized: table
      contract: {enforced: true}
    columns:
      - name: customer_id
        description: This is the primary key
        data_type: int
      - name: country_name
        description: Where this customer lives
        data_type: varchar
    
    # Declare the versions, and highlight the diffs
    versions:
    
      - v: 1
        # Matches what's above -- nothing more needed
    
      - v: 2
        # Removed a column -- this is the breaking change!
        columns:
          # This means: use the 'columns' list from above, but exclude country_name
          - include: all
            exclude: [country_name]
      
```

</File>

</TabItem>

<TabItem value="Fully specified">

<File name="models/schema.yml">

```yaml
models:
  - name: dim_customers
    latest_version: 1
    
    # declare the versions, and fully specify them
    versions:
      - v: 2
        config:
          materialized: table
          contract: {enforced: true}
        columns:
          - name: customer_id
            description: This is the primary key
            data_type: int
          # no country_name column
      
      - v: 1
        config:
          materialized: table
          contract: {enforced: true}
        columns:
          - name: customer_id
            description: This is the primary key
            data_type: int
          - name: country_name
            description: Where this customer lives
            data_type: varchar
```

</File>

</TabItem>

</Tabs>

上記の設定は、無関係な2つのモデルの代わりに、同じモデルのバージョン付き定義を2つ（`dim_customers_v1` と `dim_customers_v2`）持つことを意味します。

**これらはどこで定義されていますか？** dbt では、各モデルバージョンが `<model_name>_v<v>` という名前のファイルに定義されていることを想定しています。この場合、`dim_customers_v1.sql` と `dim_customers_v2.sql` です。追加の設定なしで、`dim_customers.sql`（サフィックスなし）に「最新」バージョンを定義することもできます。最後に、[`defined_in: any_file_name_you_want`](/reference/resource-properties/versions#defined_in) を設定することでこの規則をオーバーライドできますが、特別な理由がない限り、この規則に従うことを強くお勧めします。

**どこにマテリアライズされますか？** 各モデルバージョンは、エイリアス `<model_name>_v<v>` を持つデータベースリレーションを作成します。この場合は `dim_customers_v1` と `dim_customers_v2` です。エイリアスの設定の詳細については、[以下のセクション](#エイリアスを使用したデータベースの場所の設定) を参照してください。

**「最新」とはどのバージョンですか？** 明示的に指定されていない場合、`latest_version` は数値的に最大の `2` になります。この場合は、`latest_version: 1` と明示的に指定しています。つまり、`v2` は開発初期段階およびテスト段階の「プレリリース」です。`v2` をすべてのユーザーにデフォルトで展開する準備が整いましたら、`latest_version: 2` にアップグレードするか、仕様から `latest_version` を削除します。

### バージョン管理されたモデルの設定

各バージョンを個別に再設定できます。例えば、`v2` をテーブルとして、`v1` をビューとしてマテリアライズできます。

<File name="models/schema.yml">

```yml
versions:
  - v: 2
    config:
      materialized: table
  - v: 1
    config:
      materialized: view
```

</File>

すべての設定継承と同様に、バージョン管理されたモデルの定義 (`.sql` または `.py` ファイル) 内で設定された設定は、YAML で設定された設定よりも優先されます。

### `alias` を使用したデータベースの場所の設定

例に従って、`dim_customers_v1` で引き続き `dim_customers` というデータベーステーブルにデータを入力したいという場合を考えてみましょう。これは以前のテーブル名であり、他のダッシュボードやツールが `<dbname>.<schemaname>.dim_customers` からそのデータを読み取ることを想定している可能性があります。

`alias` 設定を使用できます:

<File name="models/schema.yml">

```yml
      - v: 1
        config:
          alias: dim_customers   # keep v1 in its original database location
```

</File>

**推奨パターン:** モデルの正規名を使用して、常に最新バージョンを指すビューまたはテーブルのクローンを作成します。このパターンに従うことで、dbt の外部でクエリを実行する場合でも、`ref` と同じ柔軟性を提供できます。特定のバージョンが必要な場合は、`_vX` サフィックスを追加してバージョン X に固定します。最新バージョンが必要な場合は、サフィックスを付けずにビューをリダイレクトします。

この機能は、すぐに使用できる機能として `dbt-core` に組み込む予定です。([dbt-core#7442](https://github.com/dbt-labs/dbt-core/issues/7442) に賛成票を投じるか、コメントを投稿してください。) 当面は、カスタムマクロと post-hook を使用して、このパターンを独自に実装できます。

<File name="macros/create_latest_version_view.sql">

```sql
{% macro create_latest_version_view() %}

    -- this hook will run only if the model is versioned, and only if it's the latest version
    -- otherwise, it's a no-op
    {% if model.get('version') and model.get('version') == model.get('latest_version') %}

        {% set new_relation = this.incorporate(path={"identifier": model['name']}) %}

        {% set existing_relation = load_relation(new_relation) %}

        {% if existing_relation and not existing_relation.is_view %}
            {{ drop_relation_if_exists(existing_relation) }}
        {% endif %}
        
        {% set create_view_sql -%}
            -- this syntax may vary by data platform
            create or replace view {{ new_relation }}
              as select * from {{ this }}
        {%- endset %}
        
        {% do log("Creating view " ~ new_relation ~ " pointing to " ~ this, info = true) if execute %}
        
        {{ return(create_view_sql) }}
        
    {% else %}
    
        -- no-op
        select 1 as id
    
    {% endif %}

{% endmacro %}
```

</File>


<File name="dbt_project.yml">

```yml
# dbt_project.yml
models:
  post-hook:
    - "{{ create_latest_version_view() }}"
```

</File>

:::info
プロジェクトでこれまで `generate_alias_name` マクロを再実装することで [カスタムエイリアス](/docs/build/custom-aliases) を実装しており、モデルバージョンの使用を開始する場合は、モデルバージョンを考慮してカスタム実装を更新する必要があります。具体的には、[次のような条件](https://github.com/dbt-labs/dbt-core/blob/ada8860e48b32ac712d92e8b0977b2c3c9749981/core/dbt/include/global_project/macros/get_custom_name/get_custom_alias.sql#L26-L30) を追加することをお勧めします。

既存の `generate_alias_name` 実装では、v1.5 への最初のアップグレード時にエラーは発生しないはずです。バージョン管理されたモデルを初めて作成するときにのみ、次のようなエラーが表示される場合があります:

```sh
dbt.exceptions.AmbiguousAliasError: Compilation Error
  dbt found two resources with the database representation "database.schema.model_name".
  dbt cannot create two resources with identical database representations. To fix this,
  change the configuration of one of these resources:
  - model.project_name.model_name.v1 (models/.../model_name.sql)
  - model.project_name.model_name.v2 (models/.../model_name_v2.sql)
```

この機能には `generate_alias_name` を使用することを選択しました。これにより、ロジックはエンド ユーザーが引き続きアクセスでき、カスタム ロジックで再実装できるようになります。
:::

### 複数のバージョンを持つモデルを実行する

複数のバージョンを持つモデルを実行するには、[`--select` フラグ](/reference/node-selection/syntax) を使用します。例:

- `dim_customers` のすべてのバージョンを実行します。

  ```bash
  dbt run --select dim_customers # Run all versions of the model
  ```
- `dim_customers` のバージョン 2 のみを実行します。

  以下のいずれかのコマンドを使用できます（どちらも同じ結果になります）。

  ```bash
    dbt run --select dim_customers.v2 # Run a specific version of the model
    dbt run --select dim_customers_v2 # Alternative syntax for the specific version
  ```

- `--select` フラグのショートカットを使用して、`dim_customers` の最新バージョンを実行します:

  ```bash
  dbt run -s dim_customers,version:latest # Run the latest version of the model
  ```

これらのコマンドにより、さまざまなバージョンの dbt モデルを柔軟に管理および実行できるようになります。

### モデルバージョンの最適化

各モデルバージョンの定義方法は完全に自由です。あるモデルのSQL定義を別のモデルにコピー＆ペーストするのは簡単ですが、バージョン間で実際に何が変更されるのかをよく考える必要があります。

例えば、新しいモデルバージョンで特定の列の名前変更や削除のみを行う場合、一方のバージョンをもう一方のバージョンの上に重ねたビューとして定義することができます:

<File name="models/dim_customers_v2.sql">

```sql
{{ config(materialized = 'view') }}

{% set dim_customers_v1 = ref('dim_customers', v=1) %}

select
{{ dbt_utils.star(from=dim_customers_v1, except=["country_name"]) }}
from {{ dim_customers_v1 }}
```

</File>

もちろん、あるモデルバージョンが別のモデルのロジックに意味のある実質的な変更を加えた場合、この方法で最適化できない可能性があります。その場合、同様の変換を再計算するコストよりも、人間の直感と可読性にかかるコストの方が重要になります。

チームが実際にモデルバージョンを採用し始めるにつれて、より明確な推奨事項が策定される予定です。私たちが想定している推奨パターンの1つは、`latest_version` の定義を優先し、最新バージョンとの差分に基づいて他のバージョン（旧バージョンとプレリリースバージョン）を定義するというものです。どのようにすればよいでしょうか？
- 最上位レベルのモデルYAMLに最新バージョンのプロパティと設定を定義し、下位レベルのモデルYAMLに他のバージョンの差分を定義します（`include`/`exclude` を使用）。
- 可能であれば、最新バージョンを起点とする `select` 変換として他のバージョンを定義します。
- `latest_version` を更新する場合は、それに応じてSQLとYAMLを移行します。

上記の例では、3番目のポイントは難しいかもしれません。 `country_name` を_除外_する方が、追加し直すよりも簡単です。代わりに、`dim_customers_v1` の元のロジックを完全に維持しつつ、それを `view` としてマテリアライズ化することで、データウェアハウスの構築コストを最小限に抑える必要があるかもしれません。下流のクエリ実行者がわずかにパフォーマンスの低下を感じたとしても、それでも壊れたクエリよりははるかに良いので、新しい「最新」バージョンに移行する理由がさらに増えます。
