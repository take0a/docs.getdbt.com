---
title: "event_time"
id: "event-time"
sidebar_label: "event_time"
resource_types: [models, seeds, source]
description: "dbtはevent_timeを使用してイベントの発生時刻を把握します。event_timeを定義すると、マイクロバッチ増分モデル、サンプルフラグ、そして高度なCIにおけるデータセットのより精密な比較が可能になります。"
datatype: string
---

<VersionCallout version="1.9" />

<Tabs>
<TabItem value="model" label="Models">

<File name='dbt_project.yml'>

```yml
models:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<File name='models/properties.yml'>

```yml
models:
  - name: model_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```
</File>

<File name="models/modelname.sql">

```sql
{{ config(
    event_time='my_time_field'
) }}
```

</File>

</TabItem>

<TabItem value="seeds" label="Seeds">

<File name='dbt_project.yml'>

```yml
seeds:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<File name='seeds/properties.yml'>

```yml
seeds:
  - name: seed_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```

</File>
</TabItem>

<TabItem value="snapshot" label="Snapshots">

<File name='dbt_project.yml'>

```yml
snapshots:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<VersionBlock firstVersion="1.9">
<File name='snapshots/properties.yml'>

```yml
snapshots:
  - name: snapshot_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```
</File>
</VersionBlock>

<VersionBlock lastVersion="1.8">

<File name="models/modlename.sql">

```sql

{{ config(
    event_time: 'my_time_field'
) }}
```

</File>


import SnapshotYaml from '/snippets.ja/_snapshot-yaml-spec.md';

<SnapshotYaml/>
</VersionBlock>



</TabItem>

<TabItem value="sources" label="Sources">

<File name='dbt_project.yml'>

```yml
sources:
  [resource-path:](/reference/resource-configs/resource-path)
    +event_time: my_time_field
```
</File>

<File name='models/properties.yml'>

```yml
sources:
  - name: source_name
    [config](/reference/resource-properties/config):
      event_time: my_time_field
```

</File>
</TabItem>
</Tabs>

## 定義

dbt は、イベントが発生したタイミングを把握するために `event_time` を使用します。`dbt_project.yml` ファイル、プロパティ YAML ファイル、または [models](/docs/build/models)、[seeds](/docs/build/seeds)、[sources](/docs/build/sources) の設定ブロックで設定してください。

### 使用方法

`event_time` は、[増分マイクロバッチ](/docs/build/incremental-microbatch) 戦略 <VersionBlock firstVersion="1.10">、[`--sample` フラグ](/docs/build/sample-flag)、</VersionBlock> に必須です。また、CI/CD ワークフローにおける [高度な CI の変更比較](/docs/deploy/advanced-ci#optimizing-comparisons) にも強く推奨されます。これにより、CI 環境と本番環境間で同じタイムスライスのデータが正しく比較されます。

### ベストプラクティス

`event_time` を、イベントの実際のタイムスタンプを表すフィールド名（`account_created_at` など）に設定してください。イベントのタイムスタンプは、イベントの取り込み日ではなく、「行が発生した時刻」を表す必要があります。`event_time` ではない列を `event_time` としてマークすると、列の意味から逸脱し、他のツールでメタデータを使用する際にユーザーの混乱を招く可能性があります。

ただし、取り込み日（`loaded_at`、`ingested_at`、`last_updated_at` など）のみを使用するタイムスタンプの場合は、これらのフィールドに `event_time` を設定できます。この場合、以下の点に注意してください。

- `last_updated_at` または `loaded_at` を使用すると、複数回実行したデータウェアハウスの結果テーブルにエントリが重複する可能性があります。適切な [lookback](/reference/resource-configs/lookback) 値を設定すると重複を減らすことができますが、ルックバック期間外の更新の一部は処理されないため、重複を完全に排除することはできません。
- `ingested_at` の使用 - この列は元のソースから取得されるのではなく、取り込み/EL ツールによって作成されるため、何らかの理由でコネクタを再同期する必要がある場合は、この列の値が変更されます。つまり、データは再処理され、別の日付でウェアハウスに再度ロードされます。このような状況が発生しない限り（または発生した場合は完全更新を実行する限り）、`ingested_at` を使用するとマイクロバッチは正しく処理されます。

推奨される `event_time` 列と推奨されない `event_time` 列の例をいくつか示します:


| <div style={{width:'200px'}}>Status</div>      | Column name     | Description    |
|--------------------|---------------------|----------------------|
| ✅ Recommended | `account_created_at` | アカウントが作成された特定の時刻を表し、時間的に固定されたイベントになります。 |
| ✅ Recommended | `session_began_at`    | ユーザー セッションが開始されたときの正確なタイムスタンプをキャプチャします。このタイムスタンプは変更されず、イベントに直接結び付けられます。 |
| ❌ Not recommended | `_fivetran_synced`    | これは、イベントが発生した時刻ではなく、イベントが取り込まれた時刻を表します。  |
| ❌ Not recommended | `last_updated_at`    | 時間の経過とともに変化し、イベント自体とは結びつきません。使用する場合は、[ベストプラクティス](#best-practices)で前述した考慮事項に注意してください。    |

## 例

<Tabs> 

<TabItem value="model" label="Models">

以下は `dbt_project.yml` ファイルの例です:

<File name='dbt_project.yml'>

```yml
models:
  my_project:
    user_sessions:
      +event_time: session_start_time
```
</File>

プロパティ YAML ファイルの例:

<File name='models/properties.yml'>

```yml
models:
  - name: user_sessions
    config:
      event_time: session_start_time
```

</File>

SQL モデル構成ブロックの例:

<File name="models/user_sessions.sql">

```sql
{{ config(
    event_time='session_start_time'
) }}
```

</File> 

この設定では、`user_sessions` モデルの `event_time` として `session_start_time` が設定されます。

</TabItem> 

<TabItem value="seeds" label="Seeds">

以下は `dbt_project.yml` ファイルの例です:

<File name='dbt_project.yml'>

```yml
seeds:
  my_project:
    my_seed:
      +event_time: record_timestamp
```

</File>

seed プロパティ YAML の例:

<File name='seeds/properties.yml'>

```yml
seeds:
  - name: my_seed
    config:
      event_time: record_timestamp
```
</File>

この設定では、`record_timestamp` が `my_seed` の `event_time` として設定されます。

</TabItem> 

<TabItem value="snapshot" label="Snapshots">

以下は `dbt_project.yml` ファイルの例です:

<File name='dbt_project.yml'>

```yml
snapshots:
  my_project:
    my_snapshot:
      +event_time: record_timestamp
```

</File>

スナップショットプロパティ YAML の例:

<File name='my_project/properties.yml'>

```yml
snapshots:
  - name: my_snapshot
    config:
      event_time: record_timestamp
```
</File>

この設定では、`record_timestamp` が `my_snapshot` の `event_time` として設定されます。

</TabItem> 

<TabItem value="sources" label="Sources">

ソース プロパティ YAML ファイルの例を次に示します:

<File name='models/properties.yml'>

```yml
sources:
  - name: source_name
    tables:
      - name: table_name
        config:
          event_time: event_timestamp
```
</File>

この設定では、指定されたソース テーブルの `event_time` として `event_timestamp` を設定します。

</TabItem> 
</Tabs>
