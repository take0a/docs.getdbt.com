---
title: "ドキュメントについて"
description: "Learn how good documentation for your dbt models helps stakeholders discover and understand your datasets."
id: "documentation"
pagination_next: "docs/build/view-documentation"
---

import CopilotBeta from '/snippets/_dbt-copilot-avail.md';

dbt モデルの適切なドキュメントは、下流の消費者が、あなたがキュレートしたデータセットを発見して理解するのに役立ちます。
dbt は、dbt プロジェクトのドキュメントを生成し、それを Web サイトとしてレンダリングする方法を提供します。


<CopilotBeta resource='documentation' />


## 関連ドキュメント

* [プロパティの宣言](/reference/configs-and-properties)
* [`dbt docs` コマンド](/reference/commands/cmd-docs)
* [`doc` Jinja 関数](/reference/dbt-jinja-functions/doc)
* dbt を初めて使用する場合は、[クイックスタート ガイド](/guides) を参照して、ドキュメントを含む最初の dbt プロジェクトを作成することをお勧めします。

## 想定される知識

* [Tests](/docs/build/data-tests)

## 概要

dbt は、説明とコマンドを使用して、dbt プロジェクトのドキュメントを [生成](#generating-documentation) するスケーラブルな方法を提供します。プロジェクトのドキュメントには、次のものが含まれます。
* **プロジェクトに関する情報**: モデル コード、プロジェクトの DAG、列に追加したテストなど。
* **<Term id="data-warehouse" /> に関する情報**: 列のデータ型、<Term id="table" /> のサイズなど。この情報は、情報スキーマに対してクエリを実行することで生成されます。
* 重要な点として、dbt では、モデル、列、ソースなどに **説明** を追加して、ドキュメントをさらに強化する方法も提供されています。

次のセクションでは、プロジェクトに [説明を追加する](#adding-descriptions-to-your-project)方法、[ドキュメントを生成する](#generating-documentation)方法、[ドキュメント ブロック](#using-docs-blocks)を使用する方法、ドキュメントの [カスタム概要](#setting-a-custom-overview)を設定する方法について説明します。

## プロジェクトに説明を追加する

ドキュメントを生成する前に、プロジェクト リソースに [descriptions](/reference/resource-properties/description) を追加します。[tests](/docs/build/data-tests) を宣言するのと同じ YAML ファイルに `description:` キーを追加します。例:

<File name='models/<filename>.yml'>

```yaml
version: 2

models:
  - name: events
    description: This table contains clickstream events from the marketing website

    columns:
      - name: event_id
        description: This is a unique identifier for the event
        tests:
          - unique
          - not_null

      - name: user-id
        quote: true
        description: The user who performed the event
        tests:
          - not_null

```
</File>


### FAQs
<FAQ path="Project/example-projects" alt_header="Are there any example dbt documentation sites?"/>
<FAQ path="Docs/document-all-columns" />
<FAQ path="Docs/long-descriptions" />
<FAQ path="Docs/sharing-documentation" />
<FAQ path="Docs/document-other-resources" />

## ドキュメントの生成

次の手順に従って、プロジェクトのドキュメントを生成します。

1. `dbt docs generate` [コマンド](/reference/commands/cmd-docs#dbt-docs-generate) を実行して、dbt プロジェクトとウェアハウスに関する関連情報をそれぞれ `manifest.json` ファイルと `catalog.json` ファイルにコンパイルします。

2. プロジェクトで説明されている列だけでなく、すべての列のドキュメントを表示するには、`dbt run` または `dbt build` を使用してモデルを作成したことを確認します。

3. ローカルで開発している場合は、`dbt docs serve` [コマンド](/reference/commands/cmd-docs#dbt-docs-serve) を実行して、これらの `.json` ファイルを使用してローカル Web サイトに入力します。

dbt では、生成されたドキュメントと説明を表示するための 2 つの補完的な方法を提供しています:

- [**dbt Docs**:](/docs/build/view-documentation#dbt-docs) モデルの系統、メタデータ、ドキュメントを含む静的ドキュメント サイト。Web サーバー (S3 や Netlify など) でホストできます。dbt Core または dbt Cloud Developer プランで利用できます。
- [**dbt Explorer**](/docs/collaborate/explore-projects): dbt Docs を基盤として、拡張メタデータ、カスタマイズ可能なビュー、より詳細なプロジェクト インサイト、コラボレーション ツールを備えた動的なリアルタイム インターフェースを提供します。dbt Cloud Team または Enterprise プランで利用できます。

dbt プロジェクトのドキュメントを最大限に活用するには、[ドキュメントの表示](/docs/build/view-documentation) を参照してください。

## ドキュメントブロックの使用

Docs ブロックは、Jinja とマークダウンを使用してモデルやその他のリソースを文書化するための堅牢な方法を提供します。Docs ブロック ファイルには任意のマークダウンを含めることができますが、一意の名前を付ける必要があります。

### 構文
ドキュメント ブロックを宣言するには、Jinja の `docs` タグを使用します。名前には次の文字を含めることができます:

- 数字で始まってはいけません
- 大文字と小文字 (A-Z、a-z)
- 数字 (0-9)
- アンダースコア (_)

<File name='events.md'>

```markdown
{% docs table_events %}

This table contains clickstream events from the marketing website.

The events in this table are recorded by [Snowplow](http://github.com/snowplow/snowplow) and piped into the warehouse on an hourly basis. The following pages of the marketing site are tracked:
 - /
 - /about
 - /team
 - /contact-us

{% enddocs %}
```

</File>

この例では、`table_events` という名前のドキュメント ブロックが、説明的なマークダウン コンテンツで定義されています。`table_events` という名前に重要な意味はありません。ドキュメント ブロックの名前には英数字とアンダースコアのみを使用し、数字で始まっていない限り、好きな名前を付けることができます。

### 配置

<VersionBlock firstVersion="1.9">

Docs ブロックは、`.md` ファイル拡張子を持つファイルに配置する必要があります。デフォルトでは、dbt はすべてのリソース パスで docs ブロックを検索します (たとえば、[model-paths](/reference/project-configs/model-paths)、[seed-paths](/reference/project-configs/seed-paths)、[analysis-paths](/reference/project-configs/analysis-paths)、[test-paths](/reference/project-configs/test-paths)、[macro-paths](/reference/project-configs/macro-paths)、および [snapshot-paths](/reference/project-configs/snapshot-paths) の組み合わせリスト)。この動作は、[docs-paths](/reference/project-configs/docs-paths) 構成を使用して調整できます。

</VersionBlock>

<VersionBlock lastVersion="1.8">

Docs blocks should be placed in files with a `.md` file extension. By default, dbt will search in all resource paths for docs blocks (for example, the combined list of [model-paths](/reference/project-configs/model-paths), [seed-paths](/reference/project-configs/seed-paths), [analysis-paths](/reference/project-configs/analysis-paths), [macro-paths](/reference/project-configs/macro-paths), and [snapshot-paths](/reference/project-configs/snapshot-paths)) &mdash; you can adjust this behavior using the [docs-paths](/reference/project-configs/docs-paths) config.

</VersionBlock>

### 使用方法
ドキュメント ブロックを使用するには、マークダウン文字列の代わりに [doc()](/reference/dbt-jinja-functions/doc) 関数を使用して、`schema.yml` ファイルから参照します。上記の例を使用すると、`table_events` ドキュメントを次のように `schema.yml` ファイルに含めることができます:

<File name='schema.yml'>

```yaml
version: 2

models:
  - name: events
    description: '{{ doc("table_events") }}'

    columns:
      - name: event_id
        description: This is a unique identifier for the event
        tests:
            - unique
            - not_null
```

</File>

結果のドキュメントでは、`'{{ doc("table_events") }}'` は、`table_events` ドキュメント ブロックで定義されたマークダウンに展開されます。


## カスタム概要の設定
*現在、dbt Docs でのみ利用可能です。*

dbt Docs Web サイトに表示される「概要」は、`__overview__` という独自のドキュメント ブロックを提供することで上書きできます。

- デフォルトでは、dbt はドキュメント サイト自体に関する役立つ情報を含む概要を提供します。
- 必要に応じて、会社のスタイル ガイド、レポートへのリンク、またはサポートの連絡先に関する情報に関する特定の情報でこのドキュメント ブロックを上書きすることをお勧めします。
- デフォルトの概要を上書きするには、次のようなドキュメント ブロックを作成します:

<File name='models/overview.md'>

```markdown
{% docs __overview__ %}
# Monthly Recurring Revenue (MRR) playbook.
This dbt project is a worked example to demonstrate how to model subscription
revenue. **Check out the full write-up \[here](https://blog.getdbt.com/modeling-subscription-revenue/),
as well as the repo for this project \[here](https://github.com/dbt-labs/mrr-playbook/).**
...

{% enddocs %}
```

</File>

### カスタム プロジェクト レベルの概要
*現在、dbt Docs でのみ利用可能です。*

`__[project_name]__` という名前のドキュメント ブロックを作成することで、ドキュメント サイトに含まれる各 dbt プロジェクト/パッケージに異なる概要を設定できます。

たとえば、閲覧者が `dbt_utils` または `snowplow` パッケージ内を移動したときに表示されるカスタムの概要ページを定義するには、次のようにします。

<File name='models/overview.md'>

```markdown
{% docs __dbt_utils__ %}
# Utility macros
Our dbt project heavily uses this suite of utility macros, especially:
- `surrogate_key`
- `test_equality`
- `pivot`
{% enddocs %}

{% docs __snowplow__ %}
# Snowplow sessionization
Our organization uses this package of transformations to roll Snowplow events
up to page views and sessions.
{% enddocs %}
```

</File>
