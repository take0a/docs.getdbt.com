---
title: freshness
description: "Read this guide to understand the `freshness` configuration in dbt."
id: "freshness"
---
# freshness <Lifecycle status="beta,managed,managed_plus" />
 
<VersionCallout version="1.10" />

<Tabs>
<TabItem value="yml" label="Project file">

<File name="dbt_project.yml">
  
```yaml
models:
  [<resource-path>](/reference/resource-configs/resource-path):
    [+](/reference/resource-configs/plus-prefix)[freshness](/reference/resource-properties/freshness):
      build_after:  # build this model no more often than every X amount of time, as long as as it has new data
        count: positive_integer
        period: minute | hour | day
        updates_on: any | all # optional config
```
  
</File>
</TabItem>

<TabItem value="project" label="Model YAML">

<File name="models/<filename>.yml">
  
```yml
models:
  - name: stg_orders
    config:
      freshness:
        build_after:  # build this model no more often than every X amount of time, as long as as it has new data
          count: positive_integer
          period: minute | hour | day
          updates_on: any | all # optional config
```
  
</File>
</TabItem>

<TabItem value="sql" label="Config block">
<File name="models/<filename>.sql">
  
```sql
{{
    config(
      freshness={
        "build_after": {     # build this model no more often than every X amount of time, as long as as it has new data
        "count": positive_integer,
        "period": "minute" | "hour" | "day",
        "updates_on": "any" | "all" # optional config
        } 
      }
    )
}}
```

</File>
</TabItem>
</Tabs>

<VersionBlock lastVersion="1.9">

This configuration is only available for the dbt Fusion engine.

</VersionBlock>

## 定義

モデルの「freshness」構成は、新しいソースデータまたは上流データが利用可能になった場合にのみモデルを再構築することで、状態認識オーケストレーションを強化します。これにより、不要な再構築を削減し、コストを最適化できます。これは、他のモデルに依存しているものの、定期的に更新するだけで済むモデルに有効です。

「freshness」は dbt ジョブオーケストレーションと連携して動作し、スケジュールされたジョブでモデルを再構築するタイミングを決定するのに役立ちます。ジョブの実行時、dbt は必要な場合にのみモデルを実行するため、モデルの不要な過剰構築を回避できます。dbt は、以下の方法でこれを行います。

- モデルに利用可能な新しいデータがあるかどうかを確認します。
- 「count」と「period」に基づいて、前回のビルドから十分な時間が経過していることを確認します。

ソースおよび上流モデル（メッシュ用）の場合、dbt はカスタムフレッシュネス計算（設定されている場合）に基づいてデータを「新規」と見なします。ソースのフレッシュネスが警告/エラーしきい値を超えると、dbt はビルド中に警告/エラーを発生させます。

構成は次の部分で構成されます。

| Configuration | Description |
|--------------|-------------|
| `build_after` | `freshness` の下にネストされた設定。新しいデータが存在する場合に、モデルが最後に構築されてから指定された count と period が経過したかどうかに基づいて、モデルを再構築するかどうかを決定するために使用されます。dbt はジョブが実行されるたびに新しいデータをチェックしますが、`build_after` により、十分な時間が経過して新しいデータが利用可能になった場合にのみモデルが再構築されます。 |
| `count` と `period` | dbt が新しいデータをチェックする頻度を指定します。たとえば、`count: 4, period: hour` は、dbt が 4 時間ごとにチェックすることを意味します。 |
| `updates_on` | オプション。上流のデータが変更されたときにジョブのビルドをトリガーするタイミングを決定します。次の値を使用します。<br /> - `any`: 最後のビルド以降、直接上流の任意のノードに新しいデータがあると、モデルがビルドされます。高速ですが、費用が増加する可能性があります。<br /> - `all`: 最後のビルド以降、直接上流の _すべて_ ノードに新しいデータがある場合にのみ、モデルがビルドされます。支出は減り、要件は増えます。 |

## デフォルト

`build_after` キーのデフォルトは次のとおりです。

```yaml
build_after:
  count: 0
  period: minute
  updates_on: any
```

つまり、デフォルトでは、新しいデータの量に関係なく、スケジュールされたジョブが実行されるたびにモデルが構築されます。

## 例

以下の例は、モデルの実行頻度を低くしたり高くしたりするための設定方法を示しています。

`freshness` YAML を設定することで、新しいデータが利用可能でない場合、かつ指定された時間間隔が経過しない限り、ビルドプロセス中にモデルをスキップすることができます。

### 低頻度

新しいデータがある限り、X 回ごとにのみビルドするようにモデルを設定することで、実行頻度を低くし（コストを削減）、モデルを構築できます。

`count: 4` と `period: hour` を指定して、モデルに `freshness` 設定を追加します。

```yaml
models:
  - name: stg_wizards
    config:
      freshness:
        build_after: 
          count: 4
          period: hour
          updates_on: all
  - name: stg_worlds
    config:
      freshness:
        build_after: 
          count: 4
          period: hour
          updates_on: all  
```

状態認識オーケストレーションジョブがトリガーされると、dbt は次の 2 つの点を確認します。

- すべての上流モデルで新しいソースデータが利用可能かどうか
- モデル `stg_wizards` と `stg_worlds` が 4 時間以上前にビルドされているかどうか

両方の条件が満たされた場合、dbt はモデルをビルドします。この場合、`updates_on: all` 構成が設定されています。`raw.wizards` ソースに新しいデータがあるものの、`stg_wizards` と `stg_worlds` が最後にビルドされてから 3 時間経過している場合は、何もビルドされません。

前の例で `updates_on: any` が設定されていた場合、`raw.wizards` ソースに新しいデータがある場合、モデルが過去 4 時間以内にビルドされていない限り、dbt はモデルをビルドします。

### より頻繁に実行

より頻繁に実行するモデルを構築したい場合（コストが増加する可能性があります）、すべての依存関係を待つのではなく、いずれかの依存関係に新しいデータが追加されたらすぐにモデルを構築するように設定できます。

モデルに「count: 1」と「period: hour」を指定して、「build_after」フレッシュネス設定を追加します。

```yaml
models:
  - name: stg_wizards
    config: 
      freshness:
        build_after: 
          count: 1
          period: hour
          updates_on: any
  - name: stg_worlds
    config:
      freshness:
        build_after: 
          count: 1
          period: hour
          updates_on: any  

```

状態認識オーケストレーションジョブが実行されると、dbt は次の 2 つの点を確認します。

- 少なくとも 1 つの上流モデルで新しいソースデータが利用可能かどうか。
- `stg_wizards` または `stg_worlds` が過去 1 時間以内にビルドされていないかどうか。

両方の条件が満たされた場合、dbt はモデルを再構築します。つまり、どちらかのモデル（`stg_wizards` または `stg_worlds`）に新しいデータがある場合、dbt はモデルを再構築します。どちらのモデルにも新しいデータがない場合は、何もビルドされません。

この例では、`updates_on: any` が設定されているため、`raw.wizards` ソースにのみ新しいデータがあり、`stg_wizards` のみが過去 1 時間以内にビルドされた（`stg_worlds` は更新されていない）場合でも、必要なのはソース更新 1 つと適切な（古い）モデル 1 つだけなので、dbt はモデルをビルドします。
