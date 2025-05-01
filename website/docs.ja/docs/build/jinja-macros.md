---
title: "Jinjaとマクロ"
description: "dbt で開発するときに、Jinja とマクロを使用して SQL を強化し、再利用可能なモジュール型ロジックを作成します。"
id: "jinja-macros"
---

## 関連リファレンスドキュメント
* [Jinja テンプレートデザイナーのドキュメント](https://jinja.palletsprojects.com/page/templates/) (外部リンク)
* [dbt Jinja コンテキスト](/reference/dbt-jinja-functions)
* [マクロプロパティ](/reference/macro-properties)

## 概要
dbt では、SQL とテンプレート言語である [Jinja](https://jinja.palletsprojects.com) を組み合わせることができます。

Jinja を使用すると、dbt プロジェクトが SQL 用のプログラミング環境となり、SQL では通常不可能なことが可能になります。
Jinja 自体はプログラミング言語ではなく、dbt プロジェクト内で SQL の機能を強化および拡張するためのツールとして機能する点に注意してください。

例えば、Jinja を使用すると、次のことが可能になります。
* SQL で制御構造（例: `if` ステートメントや `for` ループ）を使用する
* 本番環境へのデプロイメントで、dbt プロジェクトで [環境変数](/reference/dbt-jinja-functions/env_var) を使用する
* 現在のターゲットに基づいてプロジェクトのビルド方法を変更する
* あるクエリの結果を操作して別のクエリを生成します。例:
  * 支払い方法のリストを返し、支払い方法ごとに小計列を作成します (ピボット)
  * 2 つのリレーションの列のリストを返し、同じ順序で選択することで、それらを簡単に結合できるようにします
* SQL のスニペットを再利用可能な [**マクロ**](#マクロ) に抽象化します。これは、ほとんどのプログラミング言語の関数に似ています。

[`{{ ref() }}` 関数](/reference/dbt-jinja-functions/ref) を使用したことがあるなら、すでに Jinja を使っていることになります。

Jinja は、dbt プロジェクト内のあらゆる SQL で使用できます。これには、[モデル](/docs/build/sql-models)、[分析](/docs/build/analyses)、[テスト](/docs/build/data-tests)、さらには [フック](/docs/build/hooks-operations) も含まれます。

:::info Jinja とマクロを使い始める準備はできましたか?

モデルで Jinja を使用してマクロに変換する手順の例については、[Jinja の使用に関するチュートリアル](/guides/using-jinja) をご覧ください。

:::

## はじめる
### Jinja
以下は Jinja を活用した dbt モデルの例です:

<File name='/models/order_payment_method_amounts.sql'>

```sql
{% set payment_methods = ["bank_transfer", "credit_card", "gift_card"] %}

select
    order_id,
    {% for payment_method in payment_methods %}
    sum(case when payment_method = '{{payment_method}}' then amount end) as {{payment_method}}_amount,
    {% endfor %}
    sum(amount) as total_amount
from app_data.payments
group by 1
```

</File>

このクエリは次のようにコンパイルされます:

<File name='/models/order_payment_method_amounts.sql'>

```sql
select
    order_id,
    sum(case when payment_method = 'bank_transfer' then amount end) as bank_transfer_amount,
    sum(case when payment_method = 'credit_card' then amount end) as credit_card_amount,
    sum(case when payment_method = 'gift_card' then amount end) as gift_card_amount,
    sum(amount) as total_amount
from app_data.payments
group by 1
```

</File>

Jinja は、言語で使用される区切り文字（「カーリー」と呼ばれます）に基づいて識別できます:
- **式 `{{ ... }}`**: 式は、文字列を出力する場合に使用します。
式を使用して、[変数](/reference/dbt-jinja-functions/var)を参照したり、[マクロ](/docs/build/jinja-macros#macros)を呼び出したりできます。
- **文 `{% ... %}`**: 文は文字列を出力しません。
これらは、たとえば `for` ループや `if` 文を設定したり、変数を[設定](https://jinja.palletsprojects.com/en/3.1.x/templates/#assignments)または[変更](https://jinja.palletsprojects.com/en/3.1.x/templates/#expression-statement)したり、マクロを定義したりする制御フローに使用されます。
- **コメント `{# ... #}`**: Jinjaコメントは、コメント内のテキストの実行や文字列出力を防ぐために使用されます。コメントには `--` を使用しないでください。

dbt モデルで使用する場合、Jinja は有効なクエリにコンパイルされる必要があります。
Jinja がコンパイルする SQL を確認するには、次の手順に従ってください:
* **dbt Cloud を使用する場合:** コンパイルボタンをクリックすると、「コンパイル済み SQL」ペインにコンパイル済みの SQL が表示されます。
* **dbt Core を使用する場合:** コマンドラインから `dbt compile` を実行します。
次に、`target/compiled/{プロジェクト名}/` ディレクトリにあるコンパイル済みの SQL ファイルを開きます。
コードエディタで分割画面を使用して、両方のファイルを同時に開いたままにしてください。

### マクロ
Jinja の [マクロ](/docs/build/jinja-macros) は、複数回再利用できるコードです。他のプログラミング言語における「関数」に似ており、複数のモデル間でコードを繰り返す必要がある場合に非常に便利です。
マクロは `.sql` ファイルで定義され、通常は `macros` ディレクトリ ([docs](/reference/project-configs/macro-paths)) にあります。

マクロファイルには、1 つ以上のマクロを含めることができます。以下に例を示します。

<File name='macros/cents_to_dollars.sql'>

```sql

{% macro cents_to_dollars(column_name, scale=2) %}
    ({{ column_name }} / 100)::numeric(16, {{ scale }})
{% endmacro %}

```

</File>

このマクロを使用するモデルは次のようになります:

<File name='models/stg_payments.sql'>

```sql
select
  id as payment_id,
  {{ cents_to_dollars('amount') }} as amount_usd,
  ...
from app_data.payments

```

</File>

これは次のようにコンパイルされます:

<File name='target/compiled/models/stg_payments.sql'>

```sql
select
  id as payment_id,
  (amount / 100)::numeric(16, 2) as amount_usd,
  ...
from app_data.payments
```

</File>

import WhitespaceControl from '/snippets/_whitespace-control.md';

<WhitespaceControl/>

### パッケージのマクロの使用
便利なマクロが [パッケージ](/docs/build/packages) にまとめられています。最も人気のあるパッケージは [dbt-utils](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) です。

パッケージをプロジェクトにインストールすると、そのマクロをプロジェクト内で使用できるようになります。マクロを使用する際は、必ず [パッケージ名](/reference/dbt-jinja-functions/project_name) を先頭に付けてください。

```sql

select
  field_1,
  field_2,
  field_3,
  field_4,
  field_5,
  count(*)
from my_table
{{ dbt_utils.dimensions(5) }}

```

また、独自のプロジェクト内のマクロに [パッケージ名](/reference/dbt-jinja-functions/project_name) をプレフィックスとして付けることによって、そのマクロを修飾することもできます (これは主にパッケージ作成者にとって便利です)。

## FAQs

<FAQ path="Accounts/dbt-specific-jinja" />
<FAQ path="Jinja/which-jinja-docs" />
<FAQ path="Jinja/quoting-column-names" />
<FAQ path="Jinja/jinja-whitespace" />
<FAQ path="Project/debugging-jinja" />
<FAQ path="Docs/documenting-macros" />
<FAQ path="Project/why-so-many-macros" />

## dbtonic Jinja

よく書かれた Python が pythonic な (Python らしい) のと同じように、よく書かれた dbt コードは dbtonic です。

### <Term id="dry" /> 性よりも読みやすさを優先しましょう {#favor-readability-over-dry-ness}

Jinja の威力を理解すると、繰り返される行をすべてマクロに抽象化したいと思うようになるでしょう。
Jinja を使用すると、他のユーザーがモデルを解釈しにくくなる可能性があることに注意してください。Jinja と SQL を混在させる場合は、たとえ SQL の行をいくつかの場所で繰り返して記述する必要があったとしても、読みやすさを優先することをお勧めします。
すべてのモデルがマクロになっている場合は、再評価する価値があるかもしれません。

### パッケージマクロを活用する
初めてマクロを書く場合は、[dbt-utils](https://hub.getdbt.com/dbt-labs/dbt_utils/latest/) でオープンソースのマクロが公開されているかどうかを確認し、時間を節約しましょう。

### モデルの先頭で変数を設定する
`{% set ... %}` は、新しい変数を作成したり、既存の変数を更新したりするために使用できます。
変数は、インラインでハードコードするのではなく、モデルの先頭で設定することをお勧めします。
これは、可読性を高めるため、他の多くのコーディング言語から借用した手法であり、変数を2か所で参照する必要がある場合に便利です。


```sql
-- 🙅 This works, but can be hard to maintain as your code grows
{% for payment_method in ["bank_transfer", "credit_card", "gift_card"] %}
...
{% endfor %}


-- ✅ This is our preferred method of setting variables
{% set payment_methods = ["bank_transfer", "credit_card", "gift_card"] %}

{% for payment_method in payment_methods %}
...
{% endfor %}
```

<Snippet path="discourse-help-feed-header" />
<DiscourseHelpFeed tags="wee"/>
