---
title: "状態比較に関する注意事項"
description: "dbt での状態の比較に関する注意点について学びます。"
pagination_prev: "reference/node-selection/configure-state"
---

import StateModified from '/snippets.ja/_state-modified-compare.md';

[`state:` 選択メソッド](/reference/node-selection/methods#state) は強力な機能ですが、その背後には多くの複雑な要素が存在します。以下は、状態比較を活用した自動ジョブを設定する際に考慮すべき点です。

### Seeds

dbt は、サイズが 1MiB 未満のシードファイルのファイルハッシュを保存します。これらのシードの内容が変更された場合、シードは `state:modified` に含まれます。

シードファイルのサイズが 1MiB を超える場合、dbt はその内容を比較できず、警告を表示します。代わりに、dbt はシードのファイルパスのみを使用して変更を検出します。ファイルパスが変更された場合、シードは `state:modified` に含まれます。変更されていない場合は含まれません。

### Macros

dbt は、変更されたマクロに依存するリソース、または変更されたマクロに依存するマクロに依存するリソースを変更済みとしてマークします。

### Vars

モデルの定義に `var` または `env_var` が使用されている場合、dbt は現時点ではその系統を識別できず、モデルを `state:modified` に含めることができません。これは、`var` または `env_var` の値が変更されているためです。変数の変更によって構成が変化すると、モデルが変更済みとしてマークされる可能性があります。

### Tests

コマンド `dbt test -s state:modified` は、以下の両方を含みます。
- 新規/変更されたリソースから選択するテスト
- それ自体が新規または変更されたテスト

テストの追加または変更と、テストの選択対象となるリソース（モデル、シード、スナップショット）の追加または変更を同時に行う限り、すべてが「シンプルな」状態選択で期待どおりに動作するはずです。

```shell
dbt run -s "state:modified"
dbt test -s "state:modified"
```

ただし、これは複雑になる可能性があります。基盤となるモデルを変更せずに新しいテストを追加したり、新しいモデルと変更されていない古いモデルの両方から選択するテストを追加したりする場合は、モデルを事前に実行せずにテストする必要があるかもしれません。

テスト時に上流の参照を遅延させることができます。たとえば、テストが現在の環境にデータベースオブジェクトとして存在しないモデルから選択する場合、dbt は代わりに別の環境（状態マニフェストで定義されている環境）を参照します。これにより、クエリが失敗するリスクなしに「シンプルな」状態選択を使用できますが、複数の親を持つテストでは予期しない結果が生じる可能性があります。たとえば、変更されたモデルと変更されていないモデルをそれぞれ 1 つずつ依存する `relationships` テストがある場合、テストクエリは 2 つの異なる環境にまたがるデータから選択します。開発環境や CI でデータを制限またはサンプリングする場合、不一致が発生する可能性が高いため、参照整合性テストを行うことはあまり意味がありません。

`relationships` テストやデータテストを頻繁に使用する場合、または基盤となるモデルを変更せずにテストを追加することが多い場合は、CI ジョブの選択基準を調整することを検討してください。例えば、次のようになります:

```shell
dbt run -s "state:modified"
dbt test -s "state:modified" --exclude "test_name:relationships"
```
### `manifest.json` を上書きします

import Overwritesthemanifest from '/snippets.ja/_overwrites-the-manifest.md';

<Overwritesthemanifest />

#### おすすめ

import Recommendationoverwritesthemanifest from '/snippets.ja/_recommendation-overwriting-manifest.md'; 

<Recommendationoverwritesthemanifest />

### 誤検知

<VersionBlock firstVersion="1.9">

環境対応ロジックによる `state:modified` 選択時の誤検知を減らすには、`state_modified_compare_more_unrendered_values` [動作フラグ](/reference/global-configs/behavior-changes#behavior-change-flags) を `True` に設定します。

<StateModified features={'/snippets/_state-modified-compare.md'}/>

</VersionBlock>

<VersionBlock lastVersion="1.8">
State comparison works by identifying discrepancies between two manifests.  Those discrepancies could be the result of:

1. Changes made to a project in development
2. Env-aware logic that causes different behavior based on the `target`, env vars, etc., which can be avoided if you upgrade to dbt Core 1.9 and set the `state_modified_compare_more_unrendered_values` [behavior flag](/reference/global-configs/behavior-changes#behavior-change-flags) to `True`.

State comparison detects env-aware config in `dbt_project.yml`. This target-based config won't register as a modification:

<File name='dbt_project.yml'>

```yml
models:
  +materialized: "{{ 'table' if target.name == 'prod' else 'view' }}"
```

</File>

Of course, if the raw Jinja expression is modified, it will be marked as a modification.

Note that, as of now, this improved detection is true _only_ for `dbt_project.yml` configuration. It does not apply to:
- `.yml` resource properties (including `sources`)
- in-file `config()`

That means the following config—functionally identical to the snippet above—_will_ be marked as a modification when comparing across targets:

```sql
{{ config(
    materialized = ('table' if target.name == 'prod' else 'view')
) }}
```
</VersionBlock>

### 最後に

状態の比較は複雑です。すべての設定オプション間で結果整合性を実現するとともに、ユーザーが変更されたすべてのリソースを確実に、そして期待どおりのものだけを返すために必要な制御を提供したいと考えています。詳細については、dbtリポジトリの[「state」タグのオープンな問題](https://github.com/dbt-labs/dbt-core/issues?q=is%3Aopen+is%3Aissue+label%3Astate)をご覧ください。

## 関連ドキュメント
- [dbt における状態について](/reference/node-selection/state-selection)
- [状態選択の設定](/reference/node-selection/configure-state)
