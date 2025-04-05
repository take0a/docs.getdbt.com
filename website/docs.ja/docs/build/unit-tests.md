---
title: "ユニットテスト"
sidebar_label: "Unit tests"
description: "Implement unit tests to validate your dbt code."
search_weight: "heavy"
id: "unit-tests"
keywords:
  - unit test, unit tests, unit testing, dag
---

<VersionCallout version="1.8" />

これまで、dbt のテスト範囲は [“データ” テスト](/docs/build/data-tests) に限定されており、入力データの品質や結果のデータセットの構造を評価していました。ただし、これらのテストはモデルの構築後にしか実行できませんでした。

dbt Core v1.8 以降、dbt にユニット テストという追加のテスト タイプが導入されました。ソフトウェア プログラミングでは、ユニット テストは機能コードの小さな部分を検証しますが、ここでもほぼ同じように機能します。ユニット テストを使用すると、完全なモデルを本番環境で実現する前に、静的入力の小さなセットで SQL モデリング ロジックを検証できます。ユニット テストにより、テスト駆動開発が可能になり、開発者の効率とコードの信頼性が向上します。

## 始める前に

- 現在、SQL モデルの単体テストのみをサポートしています。
- 現在、_現在の_プロジェクトのモデルへの単体テストの追加のみをサポートしています。
- 現在、[`マテリアライズド ビュー`](/docs/build/materializations#materialized-view) マテリアライゼーションを使用するモデルの単体テストはサポートしていません。
- 現在、再帰 SQL を使用するモデルの単体テストはサポートしていません。
- 現在、イントロスペクト クエリを使用するモデルの単体テストはサポートしていません。
- モデルに複数のバージョンがある場合、デフォルトでは、モデルの *すべての* バージョンで単体テストが実行されます。詳細については、[バージョン管理されたモデルの単体テスト](/reference/resource-properties/unit-testing-versions) を参照してください。
- 単体テストは、[`models/` ディレクトリ](/reference/project-configs/model-paths) の YML ファイルで定義する必要があります。
- `join` ロジックを単体テストするには、テーブル名にエイリアスを付ける必要があります。
- コンパイル中に「ノードが見つかりません」というエラーが発生しないように、ユニット テスト構成にすべての [`ref`](/reference/dbt-jinja-functions/ref) または [`source`](/reference/dbt-jinja-functions/source) モデル参照を `input` として含めます。

#### アダプタ固有の注意事項
- ユニット テストでは、BigQuery の `STRUCT` 内のすべてのフィールドを指定する必要があります。`STRUCT` 内のフィールドのサブセットのみを使用することはできません。
- Redshift のお客様は、回避策が必要な [ユニット テストを構築する際の制限](/reference/resource-configs/redshift-configs#unit-test-limitations) に注意する必要があります。

ユニット テストのフォーマットの詳細については、[リファレンス ドキュメント](/reference/resource-properties/unit-tests) をお読みください。

### モデルにユニットテストを追加するタイミング

モデルのユニットテストを行う必要があります:
- SQL に複雑なロジックが含まれている場合:
    - Regex
    - Date math
    - Window functions
    - `case when` statements when there are many `when`s
    - Truncation
- 関数の作成と同様に、入力データを処理するためのカスタム ロジックを記述する場合。
- `min()` などの関数はウェアハウスによって広範囲にテストされているため、これらの関数の単体テストを実施することはお勧めしません。予期しない問題が発生した場合、関数自体の問題ではなく、基礎となるデータの問題が原因である可能性が高くなります。したがって、単体テストのフィクスチャ データは貴重な情報を提供しません。
- 以前にバグが報告されたロジック。
- 処理したい実際のデータではまだ見られないエッジ ケース。
- 変換ロジックをリファクタリングする前 (特にリファクタリングが重要な場合)。
- 「重要度」の高いモデル (パブリック、契約モデル、またはエクスポージャーの直接上流のモデル)。

### ユニットテストを実行するタイミング

dbt Labs では、開発環境または CI 環境でのみユニット テストを実行することを強く推奨しています。ユニット テストの入力は静的であるため、本番環境で実行する際に追加のコンピューティング サイクルを使用する必要はありません。開発環境ではテスト駆動型アプローチに使用し、CI では変更によってユニット テストが壊れないようにします。

[リソース タイプ](/reference/global-configs/resource-type) フラグ `--exclude-resource-type` または `DBT_EXCLUDE_RESOURCE_TYPES` 環境変数を使用して、本番ビルドからユニット テストを除外し、コンピューティングを節約します。

## モデルのユニットテスト

この例では、顧客の電子メールが有効かどうかを計算するフィールド `is_valid_email_address` を持つ新しい `dim_customers` モデルを作成します:

<file name='dim_customers.sql'>

```sql
with customers as (

    select * from {{ ref('stg_customers') }}

),

accepted_email_domains as (

    select * from {{ ref('top_level_email_domains') }}

),
	
check_valid_emails as (

    select
        customers.customer_id,
        customers.first_name,
        customers.last_name,
        customers.email,
	      coalesce (regexp_like(
            customers.email, '^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\\.[A-Za-z]{2,}$'
        )
        = true
        and accepted_email_domains.tld is not null,
        false) as is_valid_email_address
    from customers
		left join accepted_email_domains
        on customers.email_top_level_domain = lower(accepted_email_domains.tld)

)

select * from check_valid_emails
```
</file>

この例で提示されているロジックは、検証が難しい場合があります。このモデルに単体テストを追加して、`is_valid_email_address` ロジックが、`.` のないメール、`@` のないメール、無効なドメインからのメールなど、すべての既知のエッジ ケースをキャプチャすることを確認できます。

<file name='dbt_project.yml'> 

```yaml
unit_tests:
  - name: test_is_valid_email_address
    description: "Check my is_valid_email_address logic captures all known edge cases - emails without ., emails without @, and emails from invalid domains."
    model: dim_customers
    given:
      - input: ref('stg_customers')
        rows:
          - {email: cool@example.com,    email_top_level_domain: example.com}
          - {email: cool@unknown.com,    email_top_level_domain: unknown.com}
          - {email: badgmail.com,        email_top_level_domain: gmail.com}
          - {email: missingdot@gmailcom, email_top_level_domain: gmail.com}
      - input: ref('top_level_email_domains')
        rows:
          - {tld: example.com}
          - {tld: gmail.com}
    expect:
      rows:
        - {email: cool@example.com,    is_valid_email_address: true}
        - {email: cool@unknown.com,    is_valid_email_address: false}
        - {email: badgmail.com,        is_valid_email_address: false}
        - {email: missingdot@gmailcom, is_valid_email_address: false}

```
</file>

前の例では、インライン `dict` 形式を使用してモック データを定義していますが、インラインまたは別のフィクスチャ ファイルで `csv` または `sql` を使用することもできます。フィクスチャ ファイルを、任意の [テスト パス](/reference/project-configs/test-paths) の `fixtures` サブディレクトリに保存します。たとえば、`tests/fixtures/my_unit_test_fixture.sql` です。

`dict` または `csv` 形式を使用する場合は、関連する列のモック データのみを定義する必要があります。これにより、簡潔で具体的な単体テストを作成できます。

:::note

単体テストを実行する前に、単体テストするモデルの直接の親 (この例では、`stg_customers` と `top_level_email_domains`) がウェアハウス内に存在している必要があります。

ウェアハウスの費用を節約するために、[`--empty`](/reference/commands/build#the---empty-flag) フラグを使用してモデルの空のバージョンを構築します。

```bash

dbt run --select "stg_customers top_level_email_domains" --empty

```

あるいは、`dbt build` を使用して、系統順に次の操作を実行します。

- モデルでユニット テストを実行します。
- ウェアハウスでモデルをマテリアライズします。
- モデルでデータ テストを実行します。

:::

これで、このユニット テストを実行する準備が整いました。どの程度具体的にするかに応じて、コマンドにはいくつかのオプションがあります:

- `dbt test --select dim_customers` runs _all_ of the tests on `dim_customers`.
- `dbt test --select "dim_customers,test_type:unit"` runs all of the _unit_ tests on `dim_customers`.
- `dbt test --select test_is_valid_email_address` runs the test named `test_is_valid_email_address`.

```shell

dbt test --select test_is_valid_email_address
16:03:49  Running with dbt=1.8.0-a1
16:03:49  Registered adapter: postgres=1.8.0-a1
16:03:50  Found 6 models, 5 seeds, 4 data tests, 0 sources, 0 exposures, 0 metrics, 410 macros, 0 groups, 0 semantic models, 1 unit test
16:03:50  
16:03:50  Concurrency: 5 threads (target='postgres')
16:03:50  
16:03:50  1 of 1 START unit_test dim_customers::test_is_valid_email_address ................... [RUN]
16:03:51  1 of 1 FAIL 1 dim_customers::test_is_valid_email_address ............................ [FAIL 1 in 0.26s]
16:03:51  
16:03:51  Finished running 1 unit_test in 0 hours 0 minutes and 0.67 seconds (0.67s).
16:03:51  
16:03:51  Completed with 1 error and 0 warnings:
16:03:51  
16:03:51  Failure in unit_test test_is_valid_email_address (models/marts/unit_tests.yml)
16:03:51    

actual differs from expected:

@@ ,email           ,is_valid_email_address
→  ,cool@example.com,True→False
   ,cool@unknown.com,False
...,...             ,...


16:03:51  
16:03:51    compiled Code at models/marts/unit_tests.yml
16:03:51  
16:03:51  Done. PASS=0 WARN=0 ERROR=1 SKIP=0 TOTAL=1

```

巧妙な正規表現ステートメントは当初考えていたほど巧妙ではなく、モデルは誤って「cool@example.com」を無効なメールアドレスとしてフラグ付けしました。

正規表現ロジックを `'^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$'`（厄介なエスケープ文字）に更新し、ユニットテストを再実行すると、問題は解決します。

```shell

dbt test --select test_is_valid_email_address
16:09:11  Running with dbt=1.8.0-a1
16:09:12  Registered adapter: postgres=1.8.0-a1
16:09:12  Found 6 models, 5 seeds, 4 data tests, 0 sources, 0 exposures, 0 metrics, 410 macros, 0 groups, 0 semantic models, 1 unit test
16:09:12  
16:09:13  Concurrency: 5 threads (target='postgres')
16:09:13  
16:09:13  1 of 1 START unit_test dim_customers::test_is_valid_email_address ................... [RUN]
16:09:13  1 of 1 PASS dim_customers::test_is_valid_email_address .............................. [PASS in 0.26s]
16:09:13  
16:09:13  Finished running 1 unit_test in 0 hours 0 minutes and 0.75 seconds (0.75s).
16:09:13  
16:09:13  Completed successfully
16:09:13  
16:09:13  Done. PASS=1 WARN=0 ERROR=0 SKIP=0 TOTAL=1

```

これで、モデルを本番環境で使用する準備ができました。このユニット テストを追加することで、ウェアハウスで `dim_customers` をマテリアライズする _前に_ SQL ロジックの問題を検出でき、将来的にこのモデルの信頼性がさらに高まります。

## 増分モデルのユニットテスト

ユニット テストを構成するときに、マクロ、変数、または環境変数の出力をオーバーライドできます。これにより、増分モデルを「フル リフレッシュ」モードと「増分」モードでユニット テストできるようになります。

:::note
ユニット テストを実行したり、`dbt build` を実行したりする前に、まず増分モデルがデータベース内に存在している必要があります。ウェアハウスのコストを節約するために、[`--empty` フラグ](/reference/commands/build#the---empty-flag) を使用してモデルの空バージョンを構築します。また、必要に応じて、[`--select` フラグ](/reference/node-selection/syntax#shorthand) を使用して増分モデルのみを選択することもできます。

  ```shell
  dbt run --select "config.materialized:incremental" --empty
  ```

  コマンドを実行した後、そのモデルに対して通常の `dbt build` を実行し、ユニット テストを実行できます。
:::

増分モデルをテストする場合、期待される出力は、結果のモデル自体 (マージ/挿入後の最終テーブルがどのようになるか) ではなく、__マテリアライゼーションの結果__ (マージ/挿入されるもの) です。

たとえば、プロジェクトに増分モデルがあるとします:

<File name='my_incremental_model.sql'>

```sql

{{
    config(
        materialized='incremental'
    )
}}

select * from {{ ref('events') }}
{% if is_incremental() %}
where event_time > (select max(event_time) from {{ this }})
{% endif %}

```

</File>

増分ロジックが期待どおりに動作していることを保証するために、`my_incremental_model` で単体テストを定義できます:

```yml

unit_tests:
  - name: my_incremental_model_full_refresh_mode
    model: my_incremental_model
    overrides:
      macros:
        # unit test this model in "full refresh" mode
        is_incremental: false 
    given:
      - input: ref('events')
        rows:
          - {event_id: 1, event_time: 2020-01-01}
    expect:
      rows:
        - {event_id: 1, event_time: 2020-01-01}

  - name: my_incremental_model_incremental_mode
    model: my_incremental_model
    overrides:
      macros:
        # unit test this model in "incremental" mode
        is_incremental: true 
    given:
      - input: ref('events')
        rows:
          - {event_id: 1, event_time: 2020-01-01}
          - {event_id: 2, event_time: 2020-01-02}
          - {event_id: 3, event_time: 2020-01-03}
      - input: this 
        # contents of current my_incremental_model
        rows:
          - {event_id: 1, event_time: 2020-01-01}
    expect:
      # what will be inserted/merged into my_incremental_model
      rows:
        - {event_id: 2, event_time: 2020-01-02}
        - {event_id: 3, event_time: 2020-01-03}

```

現在、dbt フレームワークが既存のモデルにレコードを正しく挿入/マージしたかどうかを単体テストする方法はありませんが、[将来的にこれをサポートすることを検討しています](https://github.com/dbt-labs/dbt-core/issues/8664)。

## 一時的なモデルに依存するモデルのユニットテスト

一時的なモデルに依存するモデルを単体テストする場合は、その入力に `format: sql` を使用する必要があります。

```yml
unit_tests:
  - name: my_unit_test
    model: dim_customers
    given:
      - input: ref('ephemeral_model')
        format: sql
        rows: |
          select 1 as id, 'emily' as name
    expect:
      rows:
        - {id: 1, first_name: emily}
```


## ユニットテスト終了コード

ユニット テストの成功と失敗は、次の 2 つの終了コードで表されます。
- 合格 (0)
- 不合格 (1)

終了コードは、失敗したデータ テストを直接反映しないため、データ テストの成功と失敗の出力とは異なります。データ テストは、データ内の特定の条件を確認するように設計されたクエリで、失敗したテスト ケースごとに 1 行を返します (たとえば、`unique` テストの重複する値の数)。dbt は、失敗したレコードの数を失敗として報告します。一方、各ユニット テストは 1 つの「テスト ケース」を表すため、そのテスト ケース内で失敗したレコードの数に関係なく、結果は常に 0 (合格) または 1 (不合格) になります。

詳細については、[終了コード](/reference/exit-codes) を参照してください。


## 追加リソース

- [ユニット テストのリファレンス ページ](/reference/resource-properties/unit-tests)
- [モック データでサポートされているデータ形式](/reference/resource-properties/data-formats)
- [ユニット テストのバージョン管理されたモデル](/reference/resource-properties/unit-testing-versions)
- [ユニット テストの入力](/reference/resource-properties/unit-test-input)
- [ユニット テストのオーバーライド](/reference/resource-properties/unit-test-overrides)
- [プラットフォーム固有のデータ型](/reference/resource-properties/data-types)
