---
resource_types: [models,seeds,snapshots]
datatype: "{<dictionary>}"
default_value: {}
id: "grants"
---

dbt で生成するデータセットへのアクセスは、権限を使用することで管理できます。これらの権限を実装するには、各モデル、シード、またはスナップショットのリソース構成として権限を定義します。`dbt_project.yml` でプロジェクト全体に適用されるデフォルトの権限を定義し、各モデルの SQL ファイルまたは YAML ファイルでモデル固有の権限を定義します。

権限リソース構成を使用すると、ビルド時に特定の受信者セットとモデル、シード、またはスナップショットに権限を適用できます。モデル、シード、またはスナップショットのビルドが完了すると、dbt はビューまたはテーブルの権限が、構成済みの権限と完全に一致することを確認します。

dbt は、権限の更新時に最も効率的なアプローチを使用することを目指しています。これは、使用しているアダプタや、dbt が既存のオブジェクトを置き換えるか更新するかによって異なります。dbt が実行する grant ステートメントと revoke ステートメントの完全なセットは、いつでもデバッグログで確認できます。

可能な限り、権限付与はリソース構成として定義する必要がありますが、場合によっては手動で権限付与ステートメントを記述し、[フック](/docs/build/hooks-operations)を使用して実行する必要があるかもしれません。たとえば、次のような場合、フックが適している可能性があります。

* ビューやテーブル以外のデータベースオブジェクトに権限付与を適用する。
* よりきめ細かい行レベルおよび列レベルのアクセス権限を作成したり、マスキングポリシーを使用したり、将来の権限付与を適用したりする。
* dbt ではリソース構成を使用した標準サポートが提供されていない、データプラットフォームが提供するより高度な権限付与機能を活用する。
* 組み込みの権限付与機能では提供できない、より複雑な方法やカスタムの方法で権限付与を適用する。

フックの詳細については、[フックとオペレーション](/docs/build/hooks-operations) を参照してください。

## 定義

`grants` フィールドを使用して、リソースの権限または付与を設定できます。モデルを `run`、データを `seed`、またはデータセットを `snapshot` すると、dbt は `grant` ステートメントまたは `revoke` ステートメントを実行し、データベース オブジェクトに対する権限がリソースに構成した `grants` と一致することを確認します。

すべての構成と同様に、`grants` は [マニフェスト アーティファクト](/reference/artifacts/manifest-json) を含む dbt プロジェクト メタデータに含まれます。

### 一般的な構文

権限付与には、2 つの主要な構成要素があります。

* **権限:** データベース内のオブジェクトに対して特定のアクションまたは一連のアクション（テーブルからのデータの選択など）を実行する権限。
* **権限付与対象者:** 付与された権限の 1 人以上の受信者。プラットフォームによっては、これらを「プリンシパル」と呼ぶこともあります。たとえば、権限付与対象者は、ユーザー、ユーザーグループ、1 人以上のユーザーが保持するロール（Snowflake）、サービスアカウント（BigQuery/GCP）などです。

## 権限の設定

`dbt_project.yml` で `grants` を設定することで、プロジェクト内のすべてのモデル、パッケージ、またはサブフォルダなど、複数のリソースに権限を一度に適用できます。また、YAML の `config:` ブロック内、または `.sql` ファイル内で、特定のリソースに対して `grants` を個別に設定することもできます。

<Tabs
  defaultValue="models"
  values={[
    { label: 'Models', value: 'models', },
    { label: 'Seeds', value: 'seeds', },
    { label: 'Snapshots', value: 'snapshots', },
  ]
}>

<TabItem value="models">

<File name='models/schema.yml'>

```yml
models:
  - name: specific_model
    config:
      grants:
        select: ['reporter', 'bi']
```

</File>

`grants` 設定は、以下の方法でも定義できます:

- `dbt_project.yml` の `models` 設定ブロック内
- モデルの SQL ファイル内の `config()` Jinja マクロ内

詳細については、[設定とプロパティ](/reference/configs-and-properties) を参照してください。

</TabItem>

<TabItem value="seeds">

<File name='seeds/schema.yml'>

```yml
seeds:
  - name: seed_name
    config:
      grants:
        select: ['reporter', 'bi']
```

</File>

`grants` 設定は、`dbt_project.yml` の `seeds` 設定ブロック内でも定義できます。詳細は [設定とプロパティ](/reference/configs-and-properties) をご覧ください。

</TabItem>

<TabItem value="snapshots">

<File name='snapshots/schema.yml'>

```yml
snapshots:
  - name: snapshot_name
    config:  
      grants:
        select: ['reporter', 'bi']
```

</File>

`grants` 設定は、以下の場所でも定義できます:

- `dbt_project.yml` の `snapshots` 設定ブロック内
- スナップショットの SQL ブロック内の `config()` Jinja マクロ内

詳細については、[設定とプロパティ](/reference/configs-and-properties) を参照してください。

</TabItem>
</Tabs>

### 権限設定の継承

同じモデルに対して、`grants` を複数の場所（`dbt_project.yml` と、より具体的な `.sql` または `.yml` ファイルなど）で設定した場合、dbt のデフォルトの動作により、具体的でない権限付与対象者のセットが、より具体的な権限付与対象者のセットに置き換えられます。この「マージして上書きする」動作により、dbt がプロジェクトを解析する際に各権限が更新されます。

例:

<File name='dbt_project.yml'>

```yml
models:
  +grants:  # In this case the + is not optional, you must include it for your project to parse.
    select: ['user_a', 'user_b']
```

</File>

<File name='models/specific_model.sql'>

```sql
{{ config(grants = {'select': ['user_c']}) }}
```

</File>

この設定の結果、`specific_model` は `select` 権限を `user_c` のみに付与するように設定されます。`specific_model` を実行すると、データベースに表示される付与された権限はこれが唯一となり、dbt のログにもこの `grant` 文が記録されます。

`specific_model` の `select` 権限を付与されている既存の権限付与対象者リストを `user_c` に `add` したい場合を考えてみましょう。リスト全体を `置き換える_ のではなく。これを行うには、権限名の前に `+`（「追加」）記号を付けます。

<File name='models/specific_model.sql'>

```sql
{{ config(grants = {'+select': ['user_c']}) }}
```

</File>

これで、モデルは `user_a`、`user_b`、そして `user_c` に select 権限を付与するようになります。

**注:**
- これは、`+` プレフィックスを含む権限に対してのみ有効です。各権限は個別に動作を制御します。`select` に加えて他の権限を付与する場合、それらの権限名に `+` プレフィックスが付いていないと、新しい権限付与対象者を「追加」するのではなく「上書き」する動作が継続されます。
- この `+` の使用法は、上書きと追加のマージ動作を制御するものであり、辞書値を持つ構成を定義するための `dbt_project.yml` 内の `+` の使用法（上記の例を参照）とは異なります。詳細については、[plus プレフィックス](https://docs.getdbt.com/reference/resource-configs/plus-prefix) を参照してください。
- `grants` は、構成のマージ動作を制御するための `+` プレフィックスをサポートする最初の構成です。現時点ではこれが唯一の機能です。もし有用性が証明されれば、将来的には新規および既存の設定にこの機能を拡張する可能性があります。

### 条件付き権限付与

他の設定と同様に、Jinja を使用することで、状況に応じて権限付与を変更することができます。例えば、prod と dev で異なる権限を付与できます。

<File name='dbt_project.yml'>

```yml
models:
  +grants:
    select: "{{ ['user_a', 'user_b'] if target.name == 'prod' else ['user_c'] }}"
```

</File>

## 権限の取り消し

dbt は、ノードに `grants` 設定がアタッチされている場合にのみ、そのノードの権限を変更します（取り消しも含みます）。例えば、`dbt_project.yml` で最初に以下の権限を指定していたとします。

<File name='dbt_project.yml'>

```yml
models:
  +grants:
    select: ['user_a', 'user_b']
```

</File>

`+grants` セクション全体を削除すると、dbt は権限管理が不要になったと認識し、何も変更しません。ノードから既存の権限をすべて取り消すには、権限付与対象者のリストを空にしてください。

    <Tabs
    defaultValue="revoke-one"
    values={[
        { label: 'Revoke from one user', value: 'revoke-one', },
        { label: 'Revoke from all users', value:'revoke-all', },
        { label: 'Stop dbt from managing grants', value:'stop-managing', },
    ]
    }>

    <TabItem value="revoke-one">
    <File name='dbt_project.yml'>

    ```yml
    models:
      +grants:
        select: ['user_b']
    ```

    </File>
    </TabItem>

    <TabItem value="revoke-all">
    <File name='dbt_project.yml'>

    ```yml
    models:
      +grants:
        select: []
    ```

    </File>
    </TabItem>

    <TabItem value="stop-managing">
    <File name='dbt_project.yml'>

    ```yml
    models:

      # this section intentionally left blank
    ```

    </File>
    </TabItem>

    </Tabs>

## 一般的な例

各権限は、単一の権限付与対象者、または複数の権限付与対象者グループに付与できます。この例では、このモデルに対する `select` 権限を `bi_user` にのみ付与し、ビジネスインテリジェンス (BI) ツールでクエリを実行できるようにしています。

<File name='models/table_model.sql'>

```sql
{{ config(materialized = 'table', grants = {
    'select': 'bi_user'
}) }}
```

</File>

dbt がこのモデルを初めて実行すると、テーブルが作成され、次のようなコードが実行されます:

```sql
grant select on schema_name.table_model to bi_user;
```

この場合、増分モデルを作成し、2 人の受信者 (`bi_user` と `reporter`) に `select` 権限を付与します。

<File name='models/incremental_model.sql'>

```sql
{{ config(materialized = 'incremental', grants = {
    'select': ['bi_user', 'reporter']
}) }}
```

</File>

dbt がこのモデルを初めて実行すると、テーブルが作成され、次のようなコードが実行されます:

```sql
grant select on schema_name.incremental_model to bi_user, reporter;
```

以降の実行では、dbt はデータベース固有の SQL を使用して、`incremental_model` にすでに付与されている権限を表示し、`revoke` または `grant` ステートメントが必要かどうかを判断します。


## データベース固有の要件と注意事項

様々な機能を説明する際に使用する用語は標準化に努めていますが、データベースごとに微妙な違いが見られる場合があります。このセクションでは、データベース固有の要件と注意事項の一部について説明します。

上記および下記の例では、「select」という権限と「another_user」という被付与者について言及しています。多くのデータベースでは、これらまたは類似の用語が使用されています。データベースによっては、権限と被付与者の構文が異なる場合がありますので、dbt で「grants」を適切な名前で設定する必要があります。

<WHCode>

<div warehouse="BigQuery">

BigQuery では、「権限」は「ロール」と呼ばれ、`roles/service.roleName` という形式になります。たとえば、モデルに対して `select` 権限を付与する代わりに、`roles/bigquery.dataViewer` 権限を付与します。

権限付与対象は、ユーザー、グループ、サービスアカウント、ドメインなどです。それぞれを接頭辞で明確に区別する必要があります。たとえば、モデルへのアクセス権限を `someone@yourcompany.com` に付与するには、`user:someone@yourcompany.com` と指定する必要があります。

詳細については、Google のドキュメントをご覧ください。
- [GCP ロールについて](https://cloud.google.com/iam/docs/understanding-roles)
- [権限付与対象のフォーマット方法](https://cloud.google.com/bigquery/docs/reference/standard-sql/data-control-language#user_list)

<Snippet path="grants-vs-access-to" />

### BigQuery の例

SQL と BigQuery を使用した権限付与:

```sql
{{ config(grants = {'roles/bigquery.dataViewer': ['user:someone@yourcompany.com']}) }}
```

BigQuery を使用してモデル スキーマに権限を付与する:

<File name='models/schema.yml'>

```yml
models:
  - name: specific_model
    config:
      grants:
        roles/bigquery.dataViewer: ['user:someone@yourcompany.com']
```

</File>

</div>

<div warehouse="Databricks">

- OSS Apache Spark / Delta Lake は `grants` をサポートしていません。
- Databricks は SQL エンドポイントで `grants` を自動的に有効化します。インタラクティブ クラスターの場合、管理者は Databricks ドキュメントに記載されている以下の 2 つの設定手順を使用して、付与機能を有効にする必要があります。
- [ワークスペースのテーブルアクセス制御を有効にする](https://docs.databricks.com/administration-guide/access-control/table-acl.html)
- [クラスターのテーブルアクセス制御を有効にする](https://docs.databricks.com/security/access-control/table-acls/table-acl.html)
- `READ_METADATA` または `USAGE` を付与するには、[post-hooks](https://docs.getdbt.com/reference/resource-configs/pre-hook-post-hook) を使用します。

</div>

<div warehouse="Redshift">

* 権限の付与/取り消しは、Redshift ユーザーに対してのみ完全にサポートされています（[グループ](https://docs.aws.amazon.com/redshift/latest/dg/r_Groups.html)または[ロール](https://docs.aws.amazon.com/redshift/latest/dg/r_roles-managing.html)はサポートされていません）。関連する問題については、[dbt-redshift#415](https://github.com/dbt-labs/dbt-redshift/issues/415) を参照してください。

</div>

<div warehouse="Snowflake">

* dbt は、追加または削除する必要がある権限を計算する際に、[`copy_grants` 構成](/reference/resource-configs/snowflake-configs#copying-grants) を考慮します。
* 権限の付与と取り消しは、Snowflake ロールに対してのみ完全にサポートされています（[データベース ロール](https://docs.snowflake.com/user-guide/security-access-control-overview#types-of-roles) はサポートされていません）。

</div>

</WHCode>
