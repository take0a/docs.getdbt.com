---
title: 構成を定義する
sidebar_label: 構成を定義する
intro_text: "dbt プロジェクトでリソースの構成を定義する方法を学びます"
description: "dbt プロジェクトでリソースの構成を定義する方法を学びます"
pagination_previous: "reference/configs-and-properties"
pagination_next: "reference/define-properties"
---

リソース タイプに応じて、次の方法で dbt プロジェクトおよびインストールされたパッケージで構成を定義できます:

<VersionBlock firstVersion="1.9">

1. `models/`、`snapshots/`、`seeds/`、`analyses`、`tests/` などのサポートされているリソースディレクトリの `.yml` ファイルで [`config` プロパティ](/reference/resource-properties/config) を使用する。
2. [`dbt_project.yml` ファイル](dbt_project.yml) の対応するリソースキー (`models:`、`snapshots:`、`tests:` など) から

</VersionBlock>

<VersionBlock lastVersion="1.8">

1. Using a [`config()` Jinja macro](/reference/dbt-jinja-functions/config) within a `model`, `snapshot`, or `test` SQL file
2. Using a [`config` property](/reference/resource-properties/config) in a `.yml` file for supported resource directories like `models/`, `snapshots/`, `seeds/`, `analyses/`, or `tests/` directory.
3. From the [`dbt_project.yml` file](dbt_project.yml), under the corresponding resource key (`models:`, `snapshots:`, `tests:`, and so on)
</VersionBlock>

## 設定の継承

最も具体的な設定が常に優先されます。これは通常、上記の順序に従います。ファイル内の `config()` ブロック --> `.yml` ファイルで定義されたプロパティ --> プロジェクトファイルで定義された設定。

注: 汎用データテストは、詳細度に関して動作が少し異なります。[テスト設定](/reference/data-test-configs) を参照してください。

プロジェクトファイル内でも、設定は階層的に適用されます。最も具体的な設定が常に優先されます。例えば、プロジェクトファイル内では、`marketing` サブディレクトリに適用された設定は、`jaffle_shop` プロジェクト全体に適用された設定よりも優先されます。モデルまたはモデルディレクトリに設定を適用するには、[リソースパス](/reference/resource-configs/resource-path) をネストされた辞書キーとして定義します。

ルート dbt プロジェクト内の設定は、インストール済みパッケージ内の設定よりも優先順位が高くなります。これにより、インストールされたパッケージの構成を上書きして、dbt の実行をより細かく制御できるようになります。

## 構成の結合

ほとんどの構成は、階層的に適用すると「上書き」されます。より具体的な値が利用可能な場合は、より具体的でない値が完全に置き換えられます。ただし、いくつかの構成ではマージ動作が異なります。
- [`tags`](/tags) は加算されます。モデルの `dbt_project.yml` でいくつかのタグが設定され、さらに `.sql` ファイルで複数のタグが適用されている場合、最終的なタグセットにはそれらすべてが含まれます。
- [`meta`](/reference/resource-configs/meta) 辞書はマージされます（より具体的なキーと値のペアは、同じキーを持つより具体的でない値を置き換えます）。
- [`pre-hook` と `post-hook`](/reference/resource-configs/pre-hook-post-hook) も加算されます。

## The `+` prefix

import PlusPrefix from '/snippets.ja/_plus-prefix.md';

<PlusPrefix />


import Example from '/snippets.ja/_configs-properties.md'  ;

<Example />
