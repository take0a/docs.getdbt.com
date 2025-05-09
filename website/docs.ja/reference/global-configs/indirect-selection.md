---
title: "間接選択"
id: "indirect-selection"
sidebar: "間接選択"
---

import IndirSelect from '/snippets.ja/_indirect-selection-definitions.md';

`dbt test` または `dbt build` に `--indirect-selection` フラグを使用して、指定したノードで実行するテストを設定します。これは CLI フラグまたは環境変数として設定できます。dbt Core では、[YAML セレクター](/reference/node-selection/yaml-selectors) または `dbt_project.yml` の `flags:` ブロックでユーザー設定を行うこともできます。このブロックはプロジェクトレベルのフラグを設定します。

すべてのフラグが設定されている場合、優先順位は次のとおりです。詳細については、[グローバル設定について](/reference/global-configs/about-global-configs) を参照してください。

1. CLI 設定
1. 環境変数
1. ユーザー設定

フラグは、`empty`、`buildable`、`cautious`、または `eager` (デフォルト) に設定できます。デフォルトでは、dbt は選択したリソースに関係するすべてのテストを間接的に選択します。これらのオプションの詳細については、[テスト選択例における間接選択](/reference/node-selection/test-selection-examples?indirect-selection-mode=eager#indirect-selection)を参照してください。

<IndirSelect features={'/snippets/indirect-selection-definitions.md'}/>

The following is a visualization of the impact `--indirect-selection` and the various flags have using three models, three tests, and `dbt build` as an example:

<DocCarousel slidesPerView={1}>

<Lightbox src src="/img/docs/reference/indirect-selection-dbt-build.png" width="85%" title="dbt build" />

<Lightbox src src="/img/docs/reference/indirect-selection-eager.png" width="85%" title="Eager (default)"/>

<Lightbox src src="/img/docs/reference/indirect-selection-buildable.png" width="85%" title="Buildable"/>

<Lightbox src src="/img/docs/reference/indirect-selection-cautious.png" width="85%" title="Cautious"/>

<Lightbox src src="/img/docs/reference/indirect-selection-empty.png" width="85%" title="Empty"/>

</DocCarousel>

たとえば、CLI 構成を使用して、選択したノードのみを参照するテストを実行できます:

<File name='Usage'>

```shell
dbt test --indirect-selection cautious
```

</File>

または、環境変数を使用して、選択したノードのみを参照するテストを実行することもできます:

<File name='Env var'>

```text

$ export DBT_INDIRECT_SELECTION=cautious
dbt run

```

</File>

また、`dbt_project.yml` プロジェクト レベルのフラグを使用して、選択したノードのみを参照するテストを実行することもできます:

<File name='dbt_project.yml'>

```yaml

flags:
  indirect_selection: cautious

```

</File>
