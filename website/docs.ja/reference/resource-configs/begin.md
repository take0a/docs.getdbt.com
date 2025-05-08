---
title: "begin"
id: "begin"
sidebar_label: "begin"
resource_types: [models]
description: "dbt は、マイクロバッチ増分モデルの開始時点を決定するために「begin」を使用します。マイクロバッチ増分モデルで定義される場合、「begin」は、モデルが初めて構築されるとき、または完全に更新されるときの下限時間として使用されます。"
datatype: string
---

<VersionCallout version="1.9" />

## 定義

`begin` 設定を、[マイクロバッチ増分モデル](/docs/build/incremental-microbatch) のデータの開始タイムスタンプ値、つまりデータがマイクロバッチモデルに関連し始める時点に設定します。

[モデル](/docs/build/models) の `begin` は、`dbt_project.yml` ファイル、プロパティ YAML ファイル、または設定ブロックで設定できます。`begin` の値は、ISO 形式の日付、日時、または [相対日付](#set-begin-to-use-relative-dates) を表す文字列である必要があります。詳細については、次のセクションの [例](#examples) をご覧ください。

## 例

次の例では、`user_sessions` モデルの `begin` 設定として `2024-01-01 00:00:00` を設定します。

#### `dbt_project.yml` ファイルの例

<File name='dbt_project.yml'>

```yml
models:
  my_project:
    user_sessions:
      +begin: "2024-01-01 00:00:00"
```
</File>

#### プロパティYAMLファイルの例

<File name='models/properties.yml'>

```yml
models:
  - name: user_sessions
    config:
      begin: "2024-01-01 00:00:00"
```

</File>

#### SQLモデル構成ブロックの例

<File name="models/user_sessions.sql">

```sql
{{ config(
    begin='2024-01-01 00:00:00'
) }}
```

</File> 

#### `begin` で相対日付を使用するように設定

`begin` で相対日付を使用するように設定するには、モジュール変数 [`modules.datetime`](/reference/dbt-jinja-functions/modules#datetime) と [`modules.pytz`](/reference/dbt-jinja-functions/modules#pytz) を使用して、昨日の日付や今週の開始日などの相対タイムスタンプを動的に指定できます。

例えば、`begin` を昨日の日付に設定するには、次のようにします:

```sql
{{
    config(
        materialized = 'incremental',
        incremental_strategy='microbatch',
        unique_key = 'run_id',
        begin=(modules.datetime.datetime.now() - modules.datetime.timedelta(1)).isoformat(),
        event_time='created_at',
        batch_size='day',
    )
}}
```
