---
title: 並列マイクロバッチ実行"
sidebar_label: "Parallel microbatch execution"
description: "Learn about the 'parallel batch execution' strategy for incremental models."
intro_text: "並列バッチ実行を使用して、マイクロバッチ モデルをより高速に処理します。"
---

マイクロバッチ戦略には、モデルをより小さく管理しやすいバッチで更新できるという利点があります。ユースケースによっては、マイクロバッチモデルを並列実行するように構成することで、バッチを順次実行する場合と比較して処理速度が向上します。

並列バッチ実行とは、複数のバッチを順番に（順次）処理するのではなく、同時に処理することで、マイクロバッチモデルの処理速度を向上させることを意味します。

dbt はほとんどの場合、バッチを並列実行できるかどうかを自動的に検出するため、この設定を行う必要はありません。ただし、[`concurrent_batches` 構成](/reference/resource-properties/concurrent_batches) はオーバーライド（ゲートではなく）として使用でき、特定のケースでバッチを並列実行するかどうかを指定できます。

たとえば、12 個のバッチを持つマイクロバッチモデルがある場合、それらのバッチを並列実行できます。具体的には、[利用可能なスレッド](/docs/running-a-dbt-project/using-threads)の数によって制限されて並列実行されます。

## 前提条件

並列実行を使用するには、以下の前提条件を満たす必要があります。

- サポートされているアダプタとしてSnowflakeを使用してください。
  - 今後、さらに多くのアダプタがサポートされる予定です。
  - 今後、さらに多くのアダプタの同時実行サポートをテストし、追加していく予定です。
- バッチを並列実行できるのは、以下の場合のみです。
  - バッチが最初のバッチではないこと。
  - バッチが最後のバッチではないこと。

## 並列バッチ実行の仕組み

[前提条件](#prerequisites) の条件をチェックした後、`concurrent_batches` 値が設定されていない場合、dbt はモデルが [`{{ this }}`](/reference/dbt-jinja-functions/this) Jinja 関数を呼び出すかどうかをインテリジェントに自動検出します。`{{ this }}` を参照している場合、`{{ this }}` は現在のモデルのデータベースを表し、同じリレーションを参照すると競合が発生するため、バッチは順次実行されます。

それ以外の場合、`{{ this }}` が検出されず（他の条件が満たされている場合）、バッチは並列実行されます。これは、[`concurrent_batches` の値を設定する](/reference/resource-properties/concurrent_batches) ことでオーバーライドできます。

## 並列実行と順次実行

並列バッチ実行と順次処理のどちらを選択するかは、ユースケースの具体的な要件によって異なります。

- 並列バッチ実行は高速ですが、バッチ実行順序に依存しないロジックが必要です。たとえば、ユーザートランザクションをバッチで処理するシステムのデータパイプラインを開発している場合、パフォーマンスを向上させるために各バッチは並列実行されます。ただし、各トランザクションの処理に使用するロジックは、バッチの実行順序や完了順序に依存してはなりません。
- 順次処理は低速ですが、マイクロバッチモデルにおける[累積メトリクス](/docs/build/cumulative)などの計算には不可欠です。シーケンシャル処理ではデータが正しい順序で処理されるため、各ステップは前のステップに基づいて構築されます。

## `concurrent_batches` を設定する

デフォルトでは、dbt はマイクロバッチモデルでバッチを並列実行できるかどうかを自動検出し、ほとんどの場合正常に動作します。ただし、すべての [条件](#前提条件) を満たしている場合は、`dbt_project.yml` またはモデルの `.sql` ファイルで [`concurrent_batches` 設定](/reference/resource-properties/concurrent_batches) を設定して並列実行または順次実行を指定することにより、dbt の検出をオーバーライドできます。

<Tabs>
<TabItem value="yaml" label="dbt_project.yml">

<File name='dbt_project.yml'>

```yaml
models:
  +concurrent_batches: true # value set to true to run batches in parallel
```

</File>
</TabItem>

<TabItem value="sql" label="my_model.sql">

<File name='models/my_model.sql'>

```sql
{{
  config(
    materialized='incremental',
    incremental_strategy='microbatch',
    event_time='session_start',
    begin='2020-01-01',
    batch_size='day',
    concurrent_batches=true, # value set to true to run batches in parallel
    ...
  )
}}

select ...
```
</File>
</TabItem>
</Tabs>
