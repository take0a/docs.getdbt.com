---
title: "DAG にスナップショットを追加する"
sidebar_label: "Snapshots"
description: "Configure snapshots in dbt to track changes to your data over time."
id: "snapshots"
---

## 関連ドキュメント
* [スナップショット設定](/reference/snapshot-configs)
* [スナップショット プロパティ](/reference/snapshot-properties)
* [`snapshot` コマンド](/reference/commands/snapshot)

## スナップショットとは何ですか?
アナリストは、多くの場合、可変テーブル内の以前のデータ状態を「過去にさかのぼって」確認する必要があります。一部のソース データ システムは、履歴データへのアクセスを可能にする方法で構築されていますが、常にそうであるとは限りません。dbt は、可変 <Term id="table" /> への変更を時間の経過とともに記録するメカニズム (**スナップショット**) を提供します。

スナップショットは、可変ソース テーブルに対して [type-2 緩やかに変化するディメンション](https://en.wikipedia.org/wiki/Slowly_changing_dimension#Type_2:_add_new_row) を実装します。これらの緩やかに変化するディメンション (または SCD) は、テーブル内の行が時間の経過とともにどのように変化するかを識別します。注文が処理されると `status` フィールドが上書きされる可能性がある `orders` テーブルがあるとします。

| id | status | updated_at |
| -- | ------ | ---------- |
| 1 | pending | 2024-01-01 |

ここで、注文が「保留中」から「発送済み」に変わったとします。同じレコードは次のようになります:

| id | status | updated_at |
| -- | ------ | ---------- |
| 1 | shipped | 2024-01-02 |

この注文は現在「発送済み」状態ですが、注文が最後に「保留中」状態だった時期に関する情報が失われています。このため、注文の発送にかかった時間を分析することが困難 (または不可能) になります。dbt はこれらの変更を「スナップショット」して、行の値が時間の経過とともにどのように変化するかを理解するのに役立ちます。前の例のスナップショット テーブルの例を次に示します:

| id | status | updated_at | dbt_valid_from | dbt_valid_to |
| -- | ------ | ---------- | -------------- | ------------ |
| 1 | pending | 2024-01-01 | 2024-01-01 | 2024-01-02 |
| 1 | shipped | 2024-01-02 | 2024-01-02 | `null` |


## スナップショットの設定

<VersionBlock lastVersion="1.8" >

In old versions of dbt Core (v1.8 and earlier), snapshots must be defined in snapshot blocks inside of your [snapshots directory](/reference/project-configs/snapshot-paths). These snapshots do not have native support for environments or deferral, making previewing changes in development difficult. 

The modern, environment-aware way to create snapshots is to define them in YAML. This requires dbt Core v1.9 or later, or to be on any [dbt Cloud release track](/docs/dbt-versions/cloud-release-tracks).

- For more information about configuring snapshots in a `.sql` file, refer to the [Legacy snapshot configurations](/reference/resource-configs/snapshots-jinja-legacy) page. 

The following example shows how to configure a snapshot using the legacy syntax:

<File name='snapshots/orders_snapshot.sql'>

```sql
{% snapshot orders_snapshot %}

{{
    config(
      target_database='analytics',
      target_schema='snapshots',
      unique_key='id',

      strategy='timestamp',
      updated_at='updated_at',
    )
}}

select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

The following table outlines the configurations available for snapshots in versions 1.8 and earlier:

| Config | Description | Required? | Example |
| ------ | ----------- | --------- | ------- |
| [target_database](/reference/resource-configs/target_database) | The database that dbt should render the snapshot table into | No | analytics |
| [target_schema](/reference/resource-configs/target_schema) | The schema that dbt should render the snapshot table into | Yes | snapshots |
| [strategy](/reference/resource-configs/strategy) | The snapshot strategy to use. One of `timestamp` or `check` | Yes | timestamp |
| [unique_key](/reference/resource-configs/unique_key) | A <Term id="primary-key" /> column or expression for the record | Yes | id |
| [check_cols](/reference/resource-configs/check_cols) | If using the `check` strategy, then the columns to check | Only if using the `check` strategy | ["status"] |
| [updated_at](/reference/resource-configs/updated_at) | If using the `timestamp` strategy, the timestamp column to compare | Only if using the `timestamp` strategy | updated_at |
| [invalidate_hard_deletes](/reference/resource-configs/invalidate_hard_deletes) | Find hard deleted records in source, and set `dbt_valid_to` current time if no longer exists | No | True |

- A number of other configurations are also supported (like `tags` and `post-hook`), check out the full list [here](/reference/snapshot-configs).
- Snapshots can be configured from both your `dbt_project.yml` file and a `config` block, check out the [configuration docs](/reference/snapshot-configs) for more information.
- Note: BigQuery users can use `target_project` and `target_dataset` as aliases for `target_database` and `target_schema`, respectively.
- Before v1.9, `target_schema` (required) and `target_database` (optional) set a fixed schema or database for snapshots, making it hard to separate dev and prod environments. In v1.9, `target_schema` became optional, allowing environment-aware snapshots. By default, snapshots now use `generate_schema_name` or `generate_database_name`, but developers can still specify a custom location using [schema](/reference/resource-configs/schema) and [database](/reference/resource-configs/database), consistent with other resource types.

### Configuration example

To add a snapshot to your project:

1. Create a file in your `snapshots` directory with a `.sql` file extension. For example, `snapshots/orders.sql`
2. Use a `snapshot` block to define the start and end of a snapshot:

<File name='snapshots/orders_snapshot.sql'>

```sql
{% snapshot orders_snapshot %}

{% endsnapshot %}
```

</File>

3. Write a `select` statement within the snapshot block (tips for writing a good snapshot query are below). This select statement defines the results that you want to snapshot over time. You can use `sources` and `refs` here.

<File name='snapshots/orders_snapshot.sql'>

```sql
{% snapshot orders_snapshot %}

select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

4. Check whether the result set of your query includes a reliable timestamp column that indicates when a record was last updated. For our example, the `updated_at` column reliably indicates record changes, so we can use the `timestamp` strategy. If your query result set does not have a reliable timestamp, you'll need to instead use the `check` strategy — more details on this below.

5. Add configurations to your snapshot using a `config` block (more details below). You can also configure your snapshot from your `dbt_project.yml` file ([docs](/reference/snapshot-configs)).

<VersionBlock lastVersion="1.8">

<File name='snapshots/orders_snapshot.sql'>

```sql
{% snapshot orders_snapshot %}

{{
    config(
      target_database='analytics',
      target_schema='snapshots',
      unique_key='id',

      strategy='timestamp',
      updated_at='updated_at',
    )
}}

select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

6. Run the `dbt snapshot` [command](/reference/commands/snapshot) — for our example, a new table will be created at `analytics.snapshots.orders_snapshot`. You can change the `target_database` configuration, the `target_schema` configuration and the name of the snapshot (as defined in `{% snapshot .. %}`) will change how dbt names this table.

</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='snapshots/orders_snapshot.sql'>

```sql
{% snapshot orders_snapshot %}

{{
    config(
      schema='snapshots',
      unique_key='id',
      strategy='timestamp',
      updated_at='updated_at',
    )
}}

select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

6. Run the `dbt snapshot` [command](/reference/commands/snapshot)  &mdash; for our example, a new table will be created at `analytics.snapshots.orders_snapshot`. The [`schema`](/reference/resource-configs/schema) config will utilize the `generate_schema_name` macro.

</VersionBlock>

```
$ dbt snapshot
Running with dbt=1.8.0

15:07:36 | Concurrency: 8 threads (target='dev')
15:07:36 |
15:07:36 | 1 of 1 START snapshot snapshots.orders_snapshot...... [RUN]
15:07:36 | 1 of 1 OK snapshot snapshots.orders_snapshot..........[SELECT 3 in 1.82s]
15:07:36 |
15:07:36 | Finished running 1 snapshots in 0.68s.

Completed successfully

Done. PASS=2 ERROR=0 SKIP=0 TOTAL=1
```

7. Inspect the results by selecting from the table dbt created. After the first run, you should see the results of your query, plus the [snapshot meta fields](#snapshot-meta-fields) as described earlier.

8. Run the `dbt snapshot` command again, and inspect the results. If any records have been updated, the snapshot should reflect this.

9. Select from the `snapshot` in downstream models using the `ref` function.

<File name='models/changed_orders.sql'>

```sql
select * from {{ ref('orders_snapshot') }}
```

</File>

10. Snapshots are only useful if you run them frequently &mdash; schedule the `snapshot` command to run regularly.

</VersionBlock>

<VersionBlock firstVersion="1.9">

YAML ファイルでスナップショットを構成して、dbt にレコードの変更を検出する方法を指示します。よりクリーンで高速、かつ一貫性のあるセットアップを実現するために、モデルとともに YAML ファイルでスナップショット構成を定義します。スナップショット YAML ファイルをモデル ディレクトリまたはスナップショット ディレクトリに配置します。

<File name='snapshots/orders_snapshot.yml'>

```yaml
snapshots:
  - name: string
    relation: relation # source('my_source', 'my_table') or ref('my_model')
    [description](/reference/resource-properties/description):  markdown_string
    config:
      [database](/reference/resource-configs/database): string
      [schema](/reference/resource-configs/schema): string
      [alias](/reference/resource-configs/alias): string
      [strategy](/reference/resource-configs/strategy): timestamp | check
      [unique_key](/reference/resource-configs/unique_key): column_name_or_expression
      [check_cols](/reference/resource-configs/check_cols): [column_name] | all
      [updated_at](/reference/resource-configs/updated_at): column_name
      [snapshot_meta_column_names](/reference/resource-configs/snapshot_meta_column_names): dictionary
      [dbt_valid_to_current](/reference/resource-configs/dbt_valid_to_current): string
      [hard_deletes](/reference/resource-configs/hard-deletes): ignore | invalidate | new_record 
```

</File>

次の表は、スナップショットに使用できる構成の概要を示しています:

| Config | Description | Required? | Example |
| ------ | ----------- | --------- | ------- |
| [database](/reference/resource-configs/database) | スナップショット用のカスタムデータベースを指定する | No | analytics |
| [schema](/reference/resource-configs/schema) | スナップショットのカスタムスキーマを指定する | No | snapshots |
| [alias](/reference/resource-configs/alias)   | スナップショットのエイリアスを指定する | No | your_custom_snapshot |
| [strategy](/reference/resource-configs/strategy) | 使用するスナップショット戦略。有効な値: `timestamp` または `check` | Yes | timestamp |
| [unique_key](/reference/resource-configs/unique_key) | レコードの <Term id="primary-key" /> 列 (文字列または配列) または式 | Yes |  `id` or `[order_id, product_id]` |
| [check_cols](/reference/resource-configs/check_cols) | `check`戦略を使用する場合、チェックする列 | `check`戦略を使用する場合のみ | ["status"] |
| [updated_at](/reference/resource-configs/updated_at) | スナップショット クエリ結果の列で、各レコードが最後に更新された日時を示します。これは、`timestamp` 戦略で使用されます。使用するデータ プラットフォームに応じて、ISO 日付文字列と UNIX エポック整数をサポートする場合があります。 | `timestamp`戦略を使用する場合のみ | updated_at |
| [dbt_valid_to_current](/reference/resource-configs/dbt_valid_to_current) | 現在のスナップショット レコードの `dbt_valid_to` の値 (将来の日付など) のカスタム インジケーターを設定します。デフォルトでは、この値は `NULL` です。設定すると、dbt はスナップショット テーブル内の現在のレコードの `dbt_valid_to` に `NULL` ではなく指定された値を使用します。| No | string |
| [snapshot_meta_column_names](/reference/resource-configs/snapshot_meta_column_names) | スナップショットのメタフィールドの名前をカスタマイズする | No | dictionary |
| [hard_deletes](/reference/resource-configs/hard-deletes) | ソースから削除された行を処理する方法を指定します。サポートされているオプションは、`ignore` (デフォルト)、`invalidate` (従来の `invalidate_hard_deletes=true` を置き換えます)、および `new_record` です。| No | string |

- v1.9 では、`target_schema` がオプションになり、スナップショットが環境を認識できるようになりました。デフォルトでは、`target_schema` または `target_database` が定義されていない場合、スナップショットは `generate_schema_name` または `generate_database_name` マクロを使用してビルドする場所を決定します。
- 開発者は、他のリソース タイプと一貫性を保ちながら、[`schema`](/reference/resource-configs/schema) および [`database`](/reference/resource-configs/database) 構成を使用してカスタムの場所を設定できます。
- その他の構成もいくつかサポートされています (たとえば、`tags` や `post-hook`)。完全なリストについては、[スナップショット構成](/reference/snapshot-configs) を参照してください。
- `dbt_project.yml` ファイルと `config` ブロックの両方からスナップショットを構成できます。詳細については、[構成ドキュメント](/reference/snapshot-configs)を参照してください。

### プロジェクトにスナップショットを追加する

プロジェクトにスナップショットを追加するには、次の手順に従います。バージョン 1.8 以前のユーザーの場合は、[レガシー スナップショット構成](/reference/resource-configs/snapshots-jinja-legacy) を参照してください。

1. `snapshots` ディレクトリに YAML ファイル `snapshots/orders_snapshot.yml` を作成し、構成の詳細を追加します。`dbt_project.yml` ファイル ([docs](/reference/snapshot-configs)) からスナップショットを構成することもできます。

    <File name='snapshots/orders_snapshot.yml'>

    ```yaml
    snapshots:
      - name: orders_snapshot
        relation: source('jaffle_shop', 'orders')
        config:
          schema: snapshots
          database: analytics
          unique_key: id
          strategy: timestamp
          updated_at: updated_at
          dbt_valid_to_current: "to_date('9999-12-31')" # Specifies that current records should have `dbt_valid_to` set to `'9999-12-31'` instead of `NULL`.

    ```
    </File>

2. スナップショットは構成に重点を置いているため、変換ロジックは最小限です。通常は、ソースからすべてのデータを選択します。変換 (フィルター、重複排除など) を適用する必要がある場合は、一時モデルを定義してスナップショット構成で参照するのがベスト プラクティスです。

    <File name="models/ephemeral_orders.sql" >

    ```yaml
    {{ config(materialized='ephemeral') }}

    select * from {{ source('jaffle_shop', 'orders') }}
    ```
    </File>

3. クエリの結果セットに、レコードが最後に更新された日時を示す信頼性の高いタイムスタンプ列が含まれているかどうかを確認します。この例では、`updated_at` 列がレコードの変更を確実に示しているため、`timestamp` 戦略を使用できます。クエリの結果セットに信頼性の高いタイムスタンプが含まれていない場合は、代わりに `check` 戦略を使用する必要があります。詳細については、以下を参照してください。

4. `dbt snapshot` [コマンド](/reference/commands/snapshot) を実行します。この例では、`analytics.snapshots.orders_snapshot` に新しいテーブルが作成されます。[`schema`](/reference/resource-configs/schema) 構成では、`generate_schema_name` マクロが使用されます。
    ```
    $ dbt snapshot
    Running with dbt=1.9.0

    15:07:36 | Concurrency: 8 threads (target='dev')
    15:07:36 |
    15:07:36 | 1 of 1 START snapshot snapshots.orders_snapshot...... [RUN]
    15:07:36 | 1 of 1 OK snapshot snapshots.orders_snapshot..........[SELECT 3 in 1.82s]
    15:07:36 |
    15:07:36 | Finished running 1 snapshots in 0.68s.

    Completed successfully

    Done. PASS=2 ERROR=0 SKIP=0 TOTAL=1
    ```

5. dbt が作成したテーブル (`analytics.snapshots.orders_snapshot`) から選択して結果を確認します。最初の実行後、クエリの結果と、後で説明する [スナップショット メタ フィールド](#snapshot-meta-fields) が表示されます。

6. `dbt snapshot` コマンドを再度実行して結果を確認します。レコードが更新されている場合は、スナップショットにそれが反映されているはずです。

7. `ref` 関数を使用して、ダウンストリーム モデルの `snapshot` から選択します。

    <File name='models/changed_orders.sql'>

    ```sql
    select * from {{ ref('orders_snapshot') }}
    ```
    </File>

8. スナップショットは頻繁に実行する場合にのみ役立ちます - `dbt snapshot` コマンドを定期的に実行するようにスケジュールします。

</VersionBlock>

### 構成のベストプラクティス

<Expandable alt_header="可能な場合はタイムスタンプ戦略を使用する">

この戦略は、列の追加と削除を `check` 戦略よりも適切に処理します。

</Expandable>


<Expandable alt_header="日付範囲のクエリを簡単にするには、dbt_valid_to_current を使用します。">

デフォルトでは、現在のレコードの `dbt_valid_to` は `NULL` です。ただし、[`dbt_valid_to_current` 設定](/reference/resource-configs/dbt_valid_to_current) (dbt Core v1.9 以降で利用可能) を設定すると、現在のレコードの `dbt_valid_to` は指定した値 (`9999-12-31` など) に設定されます。

これにより、日付範囲によるフィルタリングが簡単になります。

</Expandable>

<Expandable alt_header="ユニークキーが本当にユニークであることを確認する">

dbt は行を照合するためにこの一意のキーを使用するため、このキーが実際に一意であることを確認することが非常に重要です。ソースのスナップショットを作成する場合は、ソースに一意性テストを追加することをお勧めします ([例](https://github.com/dbt-labs/jaffle_shop/blob/8e7c853c858018180bef1756ec93e193d9958c5b/models/staging/schema.yml#L26))。
</Expandable>

<VersionBlock lastVersion="1.8">

<Expandable alt_header="Use a target_schema that is separate to your analytics schema">

Snapshots cannot be rebuilt. As such, it's a good idea to put snapshots in a separate schema so end users know they are special. From there, you may want to set different privileges on your snapshots compared to your models, and even run them as a different user (or role, depending on your warehouse) to make it very difficult to drop a snapshot unless you really want to.

</Expandable>
</VersionBlock>

<VersionBlock firstVersion="1.9">

<Expandable alt_header="モデルのスキーマとは別のスキーマを使用する">

スナップショットは再構築できません。そのため、エンドユーザーにスナップショットが特別なものであることを認識してもらうために、スナップショットを別のスキーマに配置することをお勧めします。さらに、モデルとは異なる権限をスナップショットに設定したり、別のユーザー（またはウェアハウスによってはロール）で実行したりすることで、本当に必要な場合を除き、スナップショットの削除を非常に困難にすることができます。

</Expandable>

<Expandable alt_header="スナップショットを作成する前に、一時モデルを使用してデータをクリーンアップまたは変換します。">

スナップショットを作成する前にデータのクリーンアップや変換が必要な場合は、必要な変換を適用する一時モデルまたはステージングモデルを作成します。そして、このモデルをスナップショット設定で参照します。この方法により、スナップショット定義がクリーンな状態を維持し、変換を個別にテストおよび実行できます。

</Expandable>
</VersionBlock>

### スナップショットの仕組み

[`dbt snapshot` コマンド](/reference/commands/snapshot) を実行すると、以下の処理が行われます。
* **初回実行時:** dbt は初期スナップショット テーブルを作成します。これは `select` ステートメントの結果セットで、`dbt_valid_from` や `dbt_valid_to` などの列が追加されます。すべてのレコードの `dbt_valid_to = null` または [`dbt_valid_to_current`](/reference/resource-configs/dbt_valid_to_current) (dbt Core 1.9 以降で使用可能) で指定された値 (設定されている場合) になります。
* **2 回目以降の実行時:** dbt は、変更されたレコード、または新しいレコードが作成されたかどうかを確認します。
  - 変更された既存のレコードがある場合は、`dbt_valid_to` 列が更新されます。
  - 更新されたレコードと新しいレコードがスナップショット テーブルに挿入されます。これらのレコードには、`dbt_valid_to = null` または `dbt_valid_to_current` (dbt Core v1.9 以降で使用可能) で構成された値が設定されます。

<VersionBlock firstVersion="1.9">

#### 注
- これらの列名は、[snapshot_meta_column_names](#snapshot-meta-fields) 設定を使用して、チームまたは組織の慣例に合わせてカスタマイズできます。
- `dbt_valid_to_current` 設定を使用して、現在のスナップショットレコードの `dbt_valid_to` の値にカスタムインジケーター（`9999-12-31` などの未来の日付など）を設定します。デフォルトでは、この値は `NULL` です。設定すると、dbt はスナップショットテーブル内の現在のレコードの `dbt_valid_to` に `NULL` ではなく、この指定された値を使用します。
- [`hard_deletes`](/reference/resource-configs/hard-deletes) 設定を使用して、ソースで行が「削除」されたときに新しいレコードを追加することで、ハード削除を追跡します。サポートされているオプションは `ignore`、`invalidate`、および `new_record` です。
</VersionBlock>

スナップショットは、[ref](/reference/dbt-jinja-functions/ref) 関数を使用して、参照モデルと同じ方法で下流モデルで参照できます。

## 行の変更の検出
スナップショットの「戦略」は、dbt が行の変更を認識する方法を定義します。dbt には次の 2 つの戦略が組み込まれています。
- [Timestamp](#timestamp-strategy-recommended) - `updated_at` 列を使用して行の変更の有無を判断します。
- [Check](#check-strategy) - 列リストの現在の値と履歴値を比較して、行の変更の有無を判断します。

### タイムスタンプ戦略（推奨）{#timestamp-strategy-recommended}
`timestamp` 戦略では、`updated_at` フィールドを使用して行が変更されたかどうかを判断します。行に設定された `updated_at` 列が、スナップショットを最後に実行した時点よりも新しい場合、dbt は古いレコードを無効にし、新しいレコードを記録します。タイムスタンプが変更されていない場合、dbt は何もアクションを実行しません。

`timestamp` 戦略には次の構成が必要です。

| Config | Description | Example |
| ------ | ----------- | ------- |
| updated_at | ソース行が最後に更新された日時を表す列。使用するデータプラットフォームに応じて、ISO日付文字列とUnixエポック整数がサポートされる場合があります。 | `updated_at` |

**使用例:**

<VersionBlock lastVersion="1.8">

<File name='snapshots/orders_snapshot_timestamp.sql'>

```sql
{% snapshot orders_snapshot_timestamp %}

    {{
        config(
          target_schema='snapshots',
          strategy='timestamp',
          unique_key='id',
          updated_at='updated_at',
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='snapshots/orders_snapshot.yml'>

```yaml
snapshots:
  - name: orders_snapshot_timestamp
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: id
      strategy: timestamp
      updated_at: updated_at
```
</File>
</VersionBlock>

### チェック戦略
`check` 戦略は、信頼できる `updated_at` 列を持たないテーブルに役立ちます。この戦略は、列のリストを現在の値と履歴値で比較することで機能します。これらの列のいずれかが変更されている場合、dbt は古いレコードを無効にし、新しいレコードを記録します。列の値が同一の場合、dbt は何も処理しません。

`check` 戦略には、以下の設定が必要です:

| Config | Description | Example |
| ------ | ----------- | ------- |
| check_cols | 変更をチェックする列のリスト、またはすべての列をチェックする場合は「all」 | `["name", "email"]` |

:::caution check_cols = 'all'

`check` スナップショット戦略は、`check_cols = 'all'` を指定することで、_すべての_列の変更を追跡するように設定できます。チェックする列を明示的に列挙することをお勧めします。<Term id="surrogate-key" /> を使用して、多数の列を 1 つの列にまとめることを検討してください。

:::

#### 使用例

<VersionBlock lastVersion="1.8">

<File name='snapshots/orders_snapshot_check.sql'>

```sql
{% snapshot orders_snapshot_check %}

    {{
        config(
          target_schema='snapshots',
          strategy='check',
          unique_key='id',
          check_cols=['status', 'is_cancelled'],
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

</VersionBlock>

<VersionBlock firstVersion="1.9">

<File name='snapshots/orders_snapshot.yml'>

```yaml
snapshots:
  - name: orders_snapshot_check
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: id
      strategy: check
      check_cols:
        - status
        - is_cancelled
```

</File>

</VersionBlock>

####  `updated_at` の使用例

`check` 戦略を使用する場合、dbt は `check_cols` の値を比較することで変更を追跡します。デフォルトでは、dbt はタイムスタンプを使用して `dbt_updated_at`、`dbt_valid_from`、`dbt_valid_to` フィールドを更新します。オプションで `updated_at` 列を設定できます:

- `updated_at` が設定されている場合、`check` 戦略はタイムスタンプ戦略と同様に、代わりにこの列を使用します。
- `updated_at` 値が null の場合、dbt はデフォルトで現在のタイムスタンプを使用します。

次の例は、`updated_at` で `check` 戦略を使用する方法を示しています:

```yaml
snapshots:
  - name: orders_snapshot
    relation: ref('stg_orders')
    config:
      schema: snapshots
      unique_key: order_id
      strategy: check
      check_cols:
        - status
        - is_cancelled
      updated_at: updated_at
```

この例では:

- 指定された `check_cols` の少なくとも 1 つが変更されると、スナップショットは新しい行を作成します。`updated_at` 列に値（null 以外）がある場合、スナップショットはその値を使用します。それ以外の場合は、デフォルトのタイムスタンプを使用します。
- `updated_at` が設定されていない場合、dbt は自動的に [現在のタイムスタンプを使用](#チェック戦略のサンプル結果) して変更を追跡します。
- この方法は、レコードの更新を追跡するのに `updated_at` 列を信頼できないが、行の変更が検出されるたびにスナップショットの実行時間ではなく、この列を使用したい場合に使用します。

### ハード削除（オプトイン）

<VersionBlock firstVersion="1.9">

dbt v1.9 以降では、[`hard_deletes`](/reference/resource-configs/hard-deletes) 構成が `invalidate_hard_deletes` 構成に代わり、ソースから削除された行の処理方法をより細かく制御できるようになりました。`hard_deletes` 構成は独立した戦略ではなく、任意のスナップショット戦略で使用できる追加のオプトイン機能です。

`hard_deletes` 構成には、次の 3 つのオプション/フィールドがあります:

| Field | Description |
| --------- | ----------- |
| `ignore` (default) | 削除されたレコードに対してはアクションはありません。 |
| `invalidate` | 既存の `invalidate_hard_deletes=true` と同じように動作し、削除されたレコードは `dbt_valid_to` を設定することで無効化されます。 |
| `new_record` | レコードが削除されたときに、`dbt_is_deleted` [メタフィールド](#snapshot-meta-fields) を使用して、削除されたレコードを新しい行として追跡します。 |

import HardDeletes from '/snippets/_hard-deletes.md';

<HardDeletes />

#### 使用例

<File name='snapshots/orders_snapshot.yml'>

```yaml
snapshots:
  - name: orders_snapshot_hard_delete
    relation: source('jaffle_shop', 'orders')
    config:
      schema: snapshots
      unique_key: id
      strategy: timestamp
      updated_at: updated_at
      hard_deletes: new_record  # options are: 'ignore', 'invalidate', or 'new_record'
```

</File>

この例では、`hard_deletes: new_record` 設定により、削除されたレコードに対して `dbt_is_deleted` 列が `True` に設定された新しい行が追加されます。
復元されたレコードは、`dbt_is_deleted` フィールドが `False` に設定された新しい行として追加されます。

結果のテーブルは次のようになります:

| id | status | updated_at | dbt_valid_from | dbt_valid_to | dbt_is_deleted |
| -- | ------ | ---------- | -------------- | ------------ | -------------- |
| 1  | pending | 2024-01-01 10:47 | 2024-01-01 10:47 | 2024-01-01 11:05 | False          |
| 1  | shipped | 2024-01-01 11:05 | 2024-01-01 11:05 | 2024-01-01 11:20 | False          |
| 1  | deleted | 2024-01-01 11:20 | 2024-01-01 11:20 | 2024-01-01 12:00 | True           |
| 1  | restored | 2024-01-01 12:00 | 2024-01-01 12:00 |                 | False        |

</VersionBlock>

<VersionBlock lastVersion="1.8">

Rows that are deleted from the source query are not invalidated by default. With the config option `invalidate_hard_deletes`, dbt can track rows that no longer exist. This is done by left joining the snapshot table with the source table, and filtering the rows that are still valid at that point, but no longer can be found in the source table. `dbt_valid_to` will be set to the current snapshot time.

This configuration is not a different strategy as described above, but is an additional opt-in feature. It is not enabled by default since it alters the previous behavior.

For this configuration to work with the `timestamp` strategy, the configured `updated_at` column must be of timestamp type. Otherwise, queries will fail due to mixing data types.

Note, in v1.9 and higher, the [`hard_deletes`](/reference/resource-configs/hard-deletes) config replaces the `invalidate_hard_deletes` config for better control over how to handle deleted rows from the source.

#### Example usage

<File name='snapshots/orders_snapshot_hard_delete.sql'>

```sql
{% snapshot orders_snapshot_hard_delete %}

    {{
        config(
          target_schema='snapshots',
          strategy='timestamp',
          unique_key='id',
          updated_at='updated_at',
          invalidate_hard_deletes=True,
        )
    }}

    select * from {{ source('jaffle_shop', 'orders') }}

{% endsnapshot %}
```

</File>

</VersionBlock>

## スナップショットのメタフィールド

スナップショット <Term id="table">テーブル</Term> は、ソースデータセットのクローンとして作成され、追加のメタフィールド* が追加されます。

dbt Core v1.9 以降（または [dbt Cloud の「最新」リリーストラック](/docs/dbt-versions/cloud-release-tracks) でより早く利用可能）では、次のようになります。
- これらの列名は、[`snapshot_meta_column_names`](/reference/resource-configs/snapshot_meta_column_names) 設定を使用して、チームまたは組織の慣例に合わせてカスタマイズできます。
- [`dbt_valid_to_current` 設定](/reference/resource-configs/dbt_valid_to_current) を使用して、現在のスナップショットレコードの `dbt_valid_to` の値にカスタムインジケーター（`9999-12-31` などの将来の日付など）を設定できます。デフォルトでは、この値は `NULL` です。設定すると、dbt はスナップショットテーブル内の現在のレコードの `dbt_valid_to` に `NULL` ではなくこの指定された値を使用します。
- `hard_deletes='new_record'` フィールドを使用する場合、削除されたレコードを `dbt_is_deleted` メタフィールドの新しい行として追跡するには、[`hard_deletes`](/reference/resource-configs/hard-deletes) 設定を使用します。


| Field          | <div style={{width:'250px'}}>Meaning</div> | Notes | Example|
| -------------- | ------- | ----- | ------- |
| `dbt_valid_from` | このスナップショット行が最初に挿入され、有効になったときのタイムスタンプ。 | この列は、レコードのさまざまな「バージョン」を順序付けるために使用できます。 | `snapshot_meta_column_names: {dbt_valid_from: start_date}` |
| `dbt_valid_to`   | この行が無効化されたときのタイムスタンプ。現在のレコードの場合、これはデフォルトで `NULL` または `dbt_valid_to_current` で指定された値になります。| 最新のスナップショットレコードの場合、`dbt_valid_to` は `NULL` または指定された値に設定されます。  | `snapshot_meta_column_names: {dbt_valid_to: end_date}` |
| `dbt_scd_id`     | 各スナップショット行に対して生成される一意のキー。| これは dbt によって内部的に使用されます。| `snapshot_meta_column_names: {dbt_scd_id: scd_id}` |
| `dbt_updated_at` | このスナップショット行が挿入されたときのソース レコードの `updated_at` タイムスタンプ。 | これは dbt によって内部的に使用されます。 | `snapshot_meta_column_names: {dbt_updated_at: modified_date}` |
| `dbt_is_deleted` | レコードが削除されたかどうかを示す文字列値。(削除された場合は `True`、削除されていない場合は `False`)。|`hard_deletes='new_record'` が設定されている場合に追加されます。 | `snapshot_meta_column_names: {dbt_is_deleted: is_deleted}` |

これらの列名はすべて、`snapshot_meta_column_names` 設定を使用してカスタマイズできます。詳細については、こちらの [例](/reference/resource-configs/snapshot_meta_column_names#example) を参照してください。

*各列に使用されるタイムスタンプは、使用する戦略によって微妙に異なります。

- `timestamp` 戦略の場合、設定された `updated_at` 列は、`dbt_valid_from`、`dbt_valid_to`、`dbt_updated_at` 列に入力するために使用されます。

  <Expandable alt_header="タイムスタンプ戦略のサンプル結果">

  \`2024-01-01 11:00` のスナップショットクエリ結果

  | id | status  | updated_at       |
  | -- | ------- | ---------------- |
  | 1        | pending | 2024-01-01 10:47 |

  スナップショットの結果 (`11:00` はどこにも使用されていないことに注意してください):

  | id | status  | updated_at       | dbt_valid_from   | dbt_valid_to     | dbt_updated_at   |
  | -- | ------- | ---------------- | ---------------- | ---------------- | ---------------- |
  | 1        | pending | 2024-01-01 10:47 | 2024-01-01 10:47 |                  | 2024-01-01 10:47 |

  \`2024-01-01 11:30` のクエリ結果:

  | id | status  | updated_at       |
  | -- | ------- | ---------------- |
  | 1  | shipped | 2024-01-01 11:05 |

  スナップショットの結果 (`11:30` はどこにも使用されていないことに注意してください):

  | id | status  | updated_at       | dbt_valid_from   | dbt_valid_to     | dbt_updated_at   |
  | -- | ------- | ---------------- | ---------------- | ---------------- | ---------------- |
  | 1  | pending | 2024-01-01 10:47 | 2024-01-01 10:47 | 2024-01-01 11:05 | 2024-01-01 10:47 |
  | 1  | shipped | 2024-01-01 11:05 | 2024-01-01 11:05 |                  | 2024-01-01 11:05 |

  \`hard_deletes='new_record'` を使用したスナップショットの結果:

  | id | status  | updated_at       | dbt_valid_from   | dbt_valid_to     | dbt_updated_at   | dbt_is_deleted |
  |----|---------|------------------|------------------|------------------|------------------|----------------|
  | 1  | pending | 2024-01-01 10:47 | 2024-01-01 10:47 | 2024-01-01 11:05 | 2024-01-01 10:47 | False          |
  | 1  | shipped | 2024-01-01 11:05 | 2024-01-01 11:05 | 2024-01-01 11:20 | 2024-01-01 11:05 | False          |
  | 1  | deleted | 2024-01-01 11:20 | 2024-01-01 11:20 |                  | 2024-01-01 11:20 | True           |


  </Expandable>

- `check`戦略では、各列に現在のタイムスタンプが設定されます。設定されている場合、`check`戦略はタイムスタンプ戦略と同様に、代わりに`updated_at`列を使用します。

  <Expandable alt_header="チェック戦略のサンプル結果">

  `2024-01-01 11:00` のスナップショットクエリ結果

  | id | status  |
  | -- | ------- |
  | 1  | pending |

  スナップショットの結果:

  | id | status  | dbt_valid_from   | dbt_valid_to     | dbt_updated_at   |
  | -- | ------- | ---------------- | ---------------- | ---------------- |
  | 1  | pending | 2024-01-01 11:00 |                  | 2024-01-01 11:00 |

  `2024-01-01 11:30` のクエリ結果:

  | id | status  |
  | -- | ------- |
  | 1  | shipped |

  スナップショットの結果:

  | id | status  | dbt_valid_from   | dbt_valid_to     | dbt_updated_at   |
  | --- | ------- | ---------------- | ---------------- | ---------------- |
  | 1   | pending | 2024-01-01 11:00 | 2024-01-01 11:30 | 2024-01-01 11:00 |
  | 1   | shipped | 2024-01-01 11:30 |                  | 2024-01-01 11:30 |

  `hard_deletes='new_record'` を使用したスナップショットの結果:

  | id | status  |  dbt_valid_from   | dbt_valid_to     | dbt_updated_at   | dbt_is_deleted |
  |----|---------|------------------|------------------|------------------|----------------|
  | 1  | pending |  2024-01-01 11:00 | 2024-01-01 11:30 | 2024-01-01 11:00 | False          |
  | 1  | shipped | 2024-01-01 11:30 | 2024-01-01 11:40 | 2024-01-01 11:30 | False          |
  | 1  | deleted |  2024-01-01 11:40 |                  | 2024-01-01 11:40 | True           |

  </Expandable>


## FAQs
<FAQ path="Runs/run-one-snapshot" />
<FAQ path="Runs/snapshot-frequency" />
<FAQ path="Snapshots/snapshot-schema-changes" />
<FAQ path="Snapshots/snapshot-hooks" />
<FAQ path="Accounts/configurable-snapshot-path" />
<FAQ path="Snapshots/snapshot-target-is-not-a-snapshot-table" />
