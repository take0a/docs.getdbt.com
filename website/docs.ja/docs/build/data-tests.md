---
title: "DAG にデータテストを追加する"
sidebar_label: "Data tests"
description: "Configure dbt data tests to assess the quality of your input data and ensure accuracy in resulting datasets."
pagination_next: "docs/build/unit-tests"
pagination_prev: null
search_weight: "heavy"
id: "data-tests"
keywords:
  - test, tests, testing, dag
---

import CopilotBeta from '/snippets.ja/_dbt-copilot-avail.md';

<CopilotBeta resource='data tests' />

## 関連リファレンスドキュメント
* [テストコマンド](/reference/commands/test)
* [データテストプロパティ](/reference/resource-properties/data-tests)
* [データテスト構成](/reference/data-test-configs)
* [テスト選択例](/reference/node-selection/test-selection-examples)

<VersionBlock firstVersion="1.8">

:::important

dbt v1.8 からは、[ユニット テスト](/docs/build/unit-tests) と区別するために、「テスト」は「データ テスト」と呼ばれるようになりました。YAML キー `tests:` は、`data_tests:` のエイリアスとして引き続きサポートされています。詳細については、[新しい `data_tests:` 構文](#new-data_tests-syntax) を参照してください。

:::

</VersionBlock>

## 概要

データ テストは、dbt プロジェクト内のモデルやその他のリソース (ソース、シード、スナップショットなど) について行うアサーションです。`dbt test` を実行すると、dbt はプロジェクト内の各テストが合格か不合格かを通知します。

データ テストを使用すると、生成された結果についてアサーションを行うことで、各モデルの SQL の整合性を向上させることができます。すぐに使用できる状態で、モデル内の指定された列に、null 以外の値、一意の値、または別のモデルに対応する値を持つ値 (たとえば、`order` の `customer_id` は `customers` モデルの `id` に対応します)、および指定されたリストの値のみが含まれているかどうかをテストできます。データ テストを拡張して、組織固有のビジネス ロジックに適合させることができます。選択クエリの形式でモデルについて行うことができるアサーションはすべて、データ テストに変換できます。

データ テストは、失敗したレコードのセットを返します。汎用データ テスト (旧称スキーマ テスト) は、`test` ブロックを使用して定義されます。

dbt のほとんどすべてと同様に、データ テストは SQL クエリです。特に、これらは「失敗した」レコード、つまりアサーションを反証するレコードを取得しようとする `select` ステートメントです。列がモデル内で一意であるとアサートする場合、テスト クエリは重複を選択します。列が null にならないとアサートする場合、テストは null を検索します。データ テストが失敗した行を 0 行返した場合、テストは成功し、アサーションは検証されています。

dbt でデータ テストを定義する方法は 2 つあります。
* **単一** データ テストは、最も単純な形式のテストです。失敗した行を返す SQL クエリを記述できる場合は、そのクエリを [テスト ディレクトリ](/reference/project-configs/test-paths) 内の `.sql` ファイルに保存できます。これでデータ テストになり、`dbt test` コマンドによって実行されます。
* **汎用** データ テストは、引数を受け入れるパラメーター化されたクエリです。テスト クエリは、特別な `test` ブロック ([マクロ](jinja-macros) など) で定義されます。定義したら、`.yml` ファイル全体で名前で汎用テストを参照できます。モデル、列、ソース、スナップショット、シードで定義します。dbt には 4 つの汎用データ テストが組み込まれており、これらを使用することをお勧めします。

データ テストを定義することは、出力と入力が期待どおりであることを確認する優れた方法であり、コードが変更されたときの回帰を防ぐのに役立ちます。汎用データ テストは、わずかな違いはあるものの、同様のアサーションを繰り返して使用できるため、より一般的に使用される傾向があり、dbt データ テスト スイートの大部分を占める必要があります。とはいえ、データ テストを定義する両方の方法には、それぞれに適したタイミングと場所があります。

:::tip 最初のデータテストを作成する
dbt を初めて使用する場合は、[クイックスタート ガイド](/guides) を参照して、モデルとテストを含む最初の dbt プロジェクトを構築することをお勧めします。
:::

## 特異データテスト

データ テストを定義する最も簡単な方法は、失敗したレコードを返す正確な SQL を記述することです。これらは単一の目的に使用できる 1 回限りのアサーションであるため、「単一」データ テストと呼ばれます。

これらのテストは、通常は `tests` ディレクトリ ([`test-paths` 構成](/reference/project-configs/test-paths) で定義) にある `.sql` ファイルで定義されます。モデルを作成する場合と同様に、テスト定義で Jinja (`ref` および `source` を含む) を使用できます。各 `.sql` ファイルには 1 つの `select` ステートメントが含まれ、1 つのデータ テストを定義します:

<File name='tests/assert_total_payment_amount_is_positive.sql'>

```sql
-- Refunds have a negative amount, so the total amount should always be >= 0.
-- Therefore return records where total_amount < 0 to make the test fail.
select
    order_id,
    sum(amount) as total_amount
from {{ ref('fct_payments') }}
group by 1
having total_amount < 0
```

</File>

このテストの名前は、ファイル名と同じです: `assert_total_payment_amount_is_positive`。

注:
- 単一テスト ファイル内の SQL ステートメントの末尾にあるセミコロン (;) は省略してください。セミコロンがあるとテストが失敗する可能性があります。
- tests ディレクトリに配置された単一テストは、`dbt test` の実行時に自動的に実行されます。単一テストは汎用テストまたはマクロとして扱われないため、`model_name.yml` で参照しないでください。参照するとエラーが発生します。

プロジェクト内の単一テストに説明を追加するには、`tests` ディレクトリに `.yml` ファイル (たとえば、`tests/schema.yml`) を追加し、次の内容を含めます。

<File name='tests/schema.yml'>

```yaml
version: 2
data_tests:
  - name: assert_total_payment_amount_is_positive
    description: >
      Refunds have a negative amount, so the total amount should always be >= 0.
      Therefore return records where total amount < 0 to make the test fail.

```

</File>

単一データ テストは非常に簡単なので、列またはモデルの名前を変更するだけで、同じ基本構造を繰り返し記述することになります。その時点では、テストはそれほど単一ではありません。その場合は、汎用データ テストをお勧めします。

## 一般的なデータテスト
特定のデータ テストは汎用的であり、何度でも再利用できます。汎用データ テストは、パラメーター化されたクエリを含み、引数を受け入れる `test` ブロックで定義されます。次のようになります:

```sql
{% test not_null(model, column_name) %}

    select *
    from {{ model }}
    where {{ column_name }} is null

{% endtest %}
```

`model` と `column_name` という 2 つの引数があり、クエリにテンプレート化されていることに気付くでしょう。これがテストを「汎用的」にするものです。つまり、任意の数の列に、任意の数のモデルに定義でき、dbt はそれに応じて `model` と `column_name` の値を渡します。汎用テストが定義されると、既存のモデル (またはソース、シード、スナップショット) に _property_ として追加できます。これらのプロパティは、リソースと同じディレクトリの `.yml` ファイルに追加されます。

:::info
リソースにプロパティを追加する作業を初めて行う場合は、[プロパティの宣言](/reference/configs-and-properties)に関するドキュメントを確認してください。
:::

dbt には、`unique`、`not_null`、`accepted_values`、`relationships` の 4 つの汎用データ テストがあらかじめ定義されています。`orders` モデルでこれらのテストを使用する完全な例を次に示します:

```yml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null
      - name: status
        tests:
          - accepted_values:
              values: ['placed', 'shipped', 'completed', 'returned']
      - name: customer_id
        tests:
          - relationships:
              to: ref('customers')
              field: id
```

平易な英語で言えば、これらのデータ テストは次のようになります:
* `unique`: `orders` モデルの `order_id` 列は一意である必要があります
* `not_null`: `orders` モデルの `order_id` 列には null 値が含まれていてはなりません
* `accepted_values`: `orders` の `status` 列は `'placed'`、`'shipped'`、`'completed'`、または `'returned'` のいずれかである必要があります
* `relationships`: `orders` モデルの各 `customer_id` は `customers` <Term id="table" /> の `​​id` として存在します (参照整合性とも呼ばれます)

バックグラウンドでは、dbt は汎用テスト ブロックからのパラメーター化されたクエリを使用して、各データ テストの `select` クエリを構築します。これらのクエリは、アサーションが true ではない行を返します。テストが 0 行を返す場合、アサーションは成功します。

これらのデータ テストと追加の構成 ([`severity`](/reference/resource-configs/severity) および [`tags`](/reference/resource-configs/tags) を含む) の詳細については、[リファレンス セクション](/reference/resource-properties/data-tests) を参照してください。汎用データ テストのコア ロジックを提供する Jinja マクロに説明を追加することもできます。詳細については、[汎用データ テスト ロジックに説明を追加する](/best-practices/writing-custom-generic-tests#add-description-to-generic-data-test-logic) を参照してください。

### より一般的なデータテスト

これら 4 つのテストは、開始するには十分です。すぐに、より幅広い種類のテストを使用したくなるでしょう。これは良いことです。また、パッケージから汎用データ テストをインストールしたり、独自のテストを作成して、dbt プロジェクト全体で使用 (および再利用) することもできます。詳細については、[カスタム汎用テストのガイド](/best-practices/writing-custom-generic-tests) をご覧ください。

:::info
[dbt-utils](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) や [dbt-expectations](https://hub.getdbt.com/calogica/dbt_expectations/latest/) などの一部のオープンソース パッケージでは、汎用テストが定義されています。詳細については、[packages](/docs/build/packages) のドキュメントに進んでください。
:::

### 例
プロジェクトに汎用 (または「スキーマ」) テストを追加するには:

1. `models` ディレクトリに `.yml` ファイル (例: `models/schema.yml`) を追加し、次の内容を入力します (既存のモデルの `name:` 値を調整する必要がある場合があります)

<File name='models/schema.yml'>

```yaml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique
          - not_null

```

</File>

2. [`dbt test` コマンド](/reference/commands/test)を実行します。

```
$ dbt test

Found 3 models, 2 tests, 0 snapshots, 0 analyses, 130 macros, 0 operations, 0 seed files, 0 sources

17:31:05 | Concurrency: 1 threads (target='learn')
17:31:05 |
17:31:05 | 1 of 2 START test not_null_order_order_id..................... [RUN]
17:31:06 | 1 of 2 PASS not_null_order_order_id........................... [PASS in 0.99s]
17:31:06 | 2 of 2 START test unique_order_order_id....................... [RUN]
17:31:07 | 2 of 2 PASS unique_order_order_id............................. [PASS in 0.79s]
17:31:07 |
17:31:07 | Finished running 2 tests in 7.17s.

Completed successfully

Done. PASS=2 WARN=0 ERROR=0 SKIP=0 TOTAL=2

```
3. 次のいずれかの方法で、SQL dbt が実行されていることを確認します。
   * **dbt Cloud:** [詳細] タブを確認します。
   * **dbt Core:** `target/compiled` ディレクトリを確認します。


**Unique テスト**
<Tabs
  defaultValue="compiled"
  values={[
    {label: 'Compiled SQL', value: 'compiled'},
    {label: 'Templated SQL', value: 'templated'},
  ]}>
  <TabItem value="compiled">

```sql
select *
from (

    select
        order_id

    from analytics.orders
    where order_id is not null
    group by order_id
    having count(*) > 1

) validation_errors
```

  </TabItem>
  <TabItem value="templated">

```sql
select *
from (

    select
        {{ column_name }}

    from {{ model }}
    where {{ column_name }} is not null
    group by {{ column_name }}
    having count(*) > 1

) validation_errors
```

  </TabItem>
</Tabs>

**Not null test**

<Tabs
  defaultValue="compiled"
  values={[
    {label: 'Compiled SQL', value: 'compiled'},
    {label: 'Templated SQL', value: 'templated'},
  ]}>
  <TabItem value="compiled">

```sql
select *
from analytics.orders
where order_id is null
```

  </TabItem>
  <TabItem value="templated">

```sql
select *
from {{ model }}
where {{ column_name }} is null
```

  </TabItem>
</Tabs>

## テスト失敗の保存

通常、データ テスト クエリは、実行の一環として失敗を計算します。オプションの `--store-failures` フラグ、[`store_failures`](/reference/resource-configs/store_failures)、または [`store_failures_as`](/reference/resource-configs/store_failures_as) 構成を設定すると、dbt はまずテスト クエリの結果をデータベース内のテーブルに保存し、次にそのテーブルをクエリして失敗の数を計算します。

このワークフローにより、開発中に失敗したレコードをより迅速にクエリして調べることができます:

<Lightbox src="/img/docs/building-a-dbt-project/test-store-failures.gif" title="Store test failures in the database for faster development-time debugging."/>

テストの失敗を保存することを選択した場合は、次の点に注意してください。
* テスト結果テーブルは、デフォルトでは、`dbt_test__audit` というサフィックスまたは名前の付いたスキーマに作成されます。`schema` 構成を設定することで、この値を変更できます。(スキーマの命名の詳細については、[カスタム スキーマの使用](/docs/build/custom-schemas)を参照してください。)
- テストの結果は、常に同じテストの以前の失敗を **置き換え** ます。



## 新しい `data_tests:` 構文

<VersionBlock lastVersion="1.7">

dbt バージョン 1.8 では、`tests` 構成が `data_tests` に更新されました。詳細については、ドキュメント ナビゲーション メニューからバージョン v1.8 を選択してください。

</VersionBlock>

<VersionBlock firstVersion="1.8">
  
データ テストは、これまで dbt では唯一のテスト形式として「テスト」と呼ばれていました。バージョン 1.8 でユニット テストが導入されたことで、キーの名前が `tests:` から `data_tests:` に変更されました。

dbt は下位互換性のために引き続き YML 構成ファイルで `tests:` をサポートしており、ドキュメント全体で使用されていることがあります。ただし、`tests` キーと `data_tests` キーを同じリソース (単一のモデルなど) に同時に関連付けることはできません。

<File name='models/schema.yml'>

```yml
models:
  - name: orders
    columns:
      - name: order_id
        data_tests:
          - unique
          - not_null
```

</File>

<File name='dbt_project.yml'>

```yml
data_tests:
  +store_failures: true
```

</File>


</VersionBlock>

## FAQs

<FAQ path="Tests/available-tests" />
<FAQ path="Tests/test-one-model" />
<FAQ path="Runs/failed-tests" />
<FAQ path="Tests/recommended-tests" />
<FAQ path="Tests/when-to-test" />
<FAQ path="Tests/configurable-data-test-path" />
<FAQ path="Tests/testing-sources" />
<FAQ path="Tests/custom-test-thresholds" />
<FAQ path="Tests/uniqueness-two-columns" />
