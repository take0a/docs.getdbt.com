---
title: "New concepts in the dbt Fusion engine"
id: "new-concepts"
sidebar_label: "新しい概念"
description: "New concepts and configurations you will encounter when you install the dbt Fusion engine."
pagination_next: null
pagination_prev: null
---

# 新しい概念

<VersionBlock lastVersion="1.99">

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';

<FusionBeta />

</VersionBlock>

<IntroText>

<Constant name="fusion_engine" /> は [プロジェクトの SQL を完全に理解](/blog/the-levels-of-sql-comprehension) し、方言に対応した検証や正確な列レベルの系統といった高度な機能を実現します。

これが可能なのは、コンパイル手順が <Constant name="core" /> エンジンよりも包括的だからです。<Constant name="core" /> が「コンパイル」と表現していたのは、単に「レンダリング」、つまり Jinja テンプレート文字列を SQL クエリに変換してデータベースに送信することだけを意味していました。

dbt Fusion エンジンも Jinja をレンダリングできますが、その後、プロジェクト内のレンダリングされたすべてのクエリに対して、論理プランを生成し、静的分析によって検証するという第 2 段階を実行します。この静的分析手順こそが、Fusion の新機能の基盤です。

</IntroText>

| Step | dbt Core engine | dbt Fusion engine |
|------|-----------------|--------------------|
| Jinja を SQL に変換する | ✅ | ✅ |
| 論理プランを生成し、静的に分析する | ❌ | ✅ |
| レンダリングされた SQL を実行する | ✅ | ✅ |

## レンダリング戦略

<Lightbox src="/img/fusion/annotated_steps.png" title="各ドットは、モデルの実行ステップ（レンダリング、分析、実行）を表します。数字はDAG全体のステップ順序を表します。JITステップは緑色、AOTステップは紫色です。" alignment="left" width="600px"/>

<Expandable alt_header="JIT rendering and execution (dbt Core)" is_open="true">
  <video src="/img/fusion/CoreJitRun.mp4" autoPlay loop muted style={{ width: "100%", maxWidth: 950 }} />
</Expandable>

<Constant name="core" /> は常に **Just In Time (JIT) レンダリング** を使用します。モデルをレンダリングし、ウェアハウスで実行してから、次のモデルに進みます。

<Expandable alt_header="AOT rendering, analysis and execution (dbt Fusion engine)" is_open="true">
  <video src="/img/fusion/FusionAotRun.mp4" autoPlay loop muted style={{ width: "100%", maxWidth: 950 }} />
</Expandable>

<Constant name="fusion_engine" /> は、**事前（AOT）レンダリングと分析** を_デフォルト_ に設定します。プロジェクト内のすべてのモデルをレンダリングし、各モデルの論理プランを生成して静的に分析した後、ウェアハウス内のモデルの実行を開始します。

すべてのモデルを事前にレンダリングおよび分析し、すべてが有効であることが確認された時点で実行を開始することで、<Constant name="fusion_engine" /> はウェアハウスのリソースを不必要に消費することを回避します。一方、<Constant name="core" /> のエンジンによって実行されるモデル内のSQLエラーは、実行中にデータベース自体によってのみフラグが付けられます。

### イントロスペクティブなクエリのレンダリング

AOT レンダリングの例外は、イントロスペクティブモデルです。イントロスペクティブモデルとは、レンダリングされる SQL がデータベースクエリの結果に依存するモデルです。`run_query()` や `dbt_utils.get_column_values()` などのマクロを含むモデルはイントロスペクティブです。イントロスペクションは、事前レンダリングにおいて以下の理由から問題を引き起こします。

- ほとんどのイントロスペクティブクエリは、DAG 内の以前のモデルの結果に対して実行されますが、AOT レンダリング時にはそのモデルがデータベースに存在しない可能性があります。
- モデルがデータベースに存在していても、モデルが更新されるまでは古い情報になっている可能性があります。

<Constant name="fusion_engine" /> は、**イントロスペクティブモデルに対して JIT レンダリング** に切り替え、<Constant name="core" /> と同じようにレンダリングされるようにします。

`adapter.get_columns_in_relation()` や `dbt_utils.star()` などのマクロは、検査対象の [`Relation`](/reference/dbt-classes#relation) 自体が動的でない限り、事前にレンダリングして解析できます。これは、<Constant name="fusion_engine" /> がコンパイルプロセスの一環としてスキーマをメモリに読み込むためです。

## 静的解析の原則

[静的解析](https://en.wikipedia.org/wiki/Static_program_analysis) は、開発段階でモデルがエラーなくコンパイルされた場合、デプロイ時にもコンパイルエラーなく実行されることを保証することを目的としています。しかし、イントロスペクティブクエリは、モデルがソース管理にコミットされた後にレンダリングされたクエリを変更できるため、この保証を破る可能性があります。

<Constant name="fusion_engine" /> は、単一のモデルを個別に静的に解析するだけでなく、DAG の端から端までのすべてのクエリを静的に解析できるという点で独特です。データベースでさえ、その前に置かれたクエリしか検証できません。[情報フロー理論](https://roundup.getdbt.com/i/156064124/beyond-cll-information-flow-theory-and-metadata-propagation) などの概念は、 [まだ](https://www.getdbt.com/blog/where-we-re-headed-with-the-dbt-fusion-engine) dbt プラットフォームには組み込まれていませんが、安定した入力と DAG 全体で列をトレースする機能に依存しています。

### 静的解析とイントロスペクティブクエリ

Fusion がイントロスペクティブクエリを検出すると、そのモデルはジャストインタイムレンダリングに切り替わります（前述のとおり）。イントロスペクティブモデルとそのすべての子孫モデルは、JIT 静的解析の対象となります。JIT 静的解析は、上流モデルが既にマテリアライズされた後に、ほとんどの SQL エラーを捕捉し、無効なモデルの実行を阻止するため、「安全でない」と呼んでいます。

この分類は、Fusion が解析対象と実行対象の整合性を 100% 保証できなくなったことを示しています。安全でない静的解析が問題を引き起こす可能性のある最も一般的な実例は、スタンドアロンの「dbt コンパイル」ステップです（「dbt 実行」の一部として行われるコンパイルとは異なります）。

「dbt 実行」中は、JIT レンダリングによって下流モデルのコードが最新のウェアハウス状態になるように更新されますが、スタンドアロンのコンパイルでは上流モデルは更新されません。このシナリオでは、Fusion は最後に実行された上流モデルから読み取ります。これはおそらく問題ありませんが、エラーが誤って報告される（偽陽性）か、まったく報告されない（偽陰性）可能性があります。

<Expandable alt_header="Rendering and analyzing without execution" is_open="true">
  <video src="/img/fusion/FusionJitCompileUnsafe.mp4" autoPlay loop muted style={{ width: "100%", maxWidth: 950 }} />
  `model_d` はイントロスペクションを使用しないため AOT でレンダリングされますが、`introspective_model_c` が分析されるまで待機する必要があることに注意してください。
</Expandable>

「安全でない」静的解析でも、静的解析を行わない場合と比べて大きなメリットが得られます。問題が発生することがない限り、静的解析は有効のままにしておくことをお勧めします。さらに良い方法としては、イントロスペクティブコードをAOTレンダリングと静的解析に適した方法で書き換えられるかどうかを検討してください。

## エンジン間の違いをまとめる

dbt Core:

- すべてのモデルをジャストインタイムでレンダリングします。
- 静的解析は実行しません。

dbt Fusion エンジン:

- イントロスペクティブクエリを使用しない限り、すべてのモデルをアヘッドオブタイムでレンダリングします。
- すべてのモデルを静的に解析します。モデル自身またはその親がジャストインタイムでレンダリングされていない限り、デフォルトでアヘッドオブタイムで実行されます。親がジャストインタイムでレンダリングされている場合は、静的解析ステップもジャストインタイムで実行されます。

## `static_analysis` の設定

上記のデフォルトの動作に加えて、プロジェクト内の特定のモデルに対して静的解析を適用する方法をいつでも変更できます。**モデルが静的解析の対象となるのは、そのすべての親モデルも静的解析の対象となっている場合のみです。**

`static_analysis` のオプションは次のとおりです。

- `on`: SQL を静的に解析します。非イントロスペクトモデルの場合のデフォルトです。AOT レンダリングに依存します。
- `unsafe`: SQL を静的に解析します。イントロスペクトモデルの場合のデフォルトです。常に JIT レンダリングを使用します。
- `off`: このモデルとその子孫モデルに対して SQL 解析をスキップします。

静的解析を無効にすると、SQL 理解に依存する VS Code 拡張機能の機能が使用できなくなります。

`static_analysis` を設定するのに最適な場所は、個々のモデルまたはモデルグループの構成ファイルです。デバッグを支援するために、CLI フラグ `--static-analysis off` または `--static-analysis unsafe` を使用して、すべてのモデルレベルの設定をオーバーライドすることもできます。設定の詳細については、[CLI オプション](/reference/global-configs/command-line-options) および [設定とプロパティ](/reference/configs-and-properties) を参照してください。

### 設定例

パッケージ内のすべてのモデルの静的解析を無効にする:

<File name='dbt_project.yml'>

```yml
name: jaffle_shop

models:
  jaffle_shop: 
    marts:
      +materialized: table
  
  a_package_with_introspective_queries:
    +static_analysis: off
```

</File>

YAML で静的分析を無効にする:

<File name='models/my_udf_using_model.yml'>

```yml
models:
  - name: model_with_static_analysis_off
    config:
      static_analysis: off
```

</File>


カスタム UDF を使用してモデルの静的解析を無効にします。

<File name='models/my_udf_using_model.sql'>

```sql
{{ config(static_analysis='off') }}

select 
  user_id,
  my_cool_udf(ip_address) as cleaned_ip
from {{ ref('my_model') }}
```

</File>

### 静的解析を「オフ」にするのはどのような場合ですか？

有効なクエリに以下のものが含まれている場合、静的解析が誤って失敗することがあります。

- <Constant name="fusion_engine" /> が認識しない **構文またはネイティブ関数**。静的解析を無効にするだけでなく、[問題を開く](https://github.com/dbt-labs/dbt-fusion/issues)も行ってください。
- <Constant name="fusion_engine" /> が認識しない **ユーザー定義関数**。静的解析を一時的に無効にする必要があります。UDF コンパイルのネイティブサポートは将来のバージョンで提供される予定です。[dbt-fusion#69](https://github.com/dbt-labs/dbt-fusion/issues/69) を参照してください。
- **動的SQL** ([SnowflakeのPIVOT ANY](https://docs.snowflake.com/en/sql-reference/constructs/pivot#dynamic-pivot-on-all-distinct-column-values-automatically)など)は静的に分析できません。静的分析を無効にするか、明示的な列名を使用するようにピボットをリファクタリングするか、[Jinjaで動的ピボット](https://github.com/dbt-labs/dbt-utils#pivot-source)を作成することができます。
- **イントロスペクティブクエリに入力される揮発性の高いデータ** (スタンドアロンの`dbt compile`呼び出し中)。`dbt compile`ステップではモデルが実行されないため、イントロスペクティブクエリの実行時に古いデータを使用するか、別の環境に依存します。入力データの変更頻度が高いほど、この差異によってコンパイルエラーが発生する可能性が高くなります。静的分析を無効にする前に、これらのスタンドアロンの `dbt compile` コマンドが必要かどうかを検討してください。

## 例

### イントロスペクティブなモデルはない
<Expandable alt_header="AOT rendering, analysis and execution" is_open="true">
  <video src="/img/fusion/FusionAotRun.mp4" autoPlay loop muted style={{ width: "100%", maxWidth: 950 }} />
</Expandable>

- Fusion は各モデルを順番にレンダリングします。
- 次に、各モデルの論理プランを順番に静的に解析します。
- 最後に、各モデルのレンダリングされた SQL を実行します。Fusion がプロジェクト全体を検証するまで、データベースには何も保存されません。

### `unsafe` 静的解析を含むイントロスペクティブモデル

`model_c` を更新し、イントロスペクティブクエリ（`dbt_utils.get_column_values` など）を追加するとします。`model_b` に対してクエリを実行していると仮定しますが、<Constant name="fusion_engine" /> のレスポンスは、イントロスペクションの実行内容に関わらず同じです。

<Expandable alt_header="Unsafe static analysis of introspective models" is_open="true">
  <video src="/img/fusion/FusionJitRunUnsafe.mp4" autoPlay loop muted style={{ width: "100%", maxWidth: 950 }} />
</Expandable>

- 解析中に、Fusion は `model_c` のイントロスペクションクエリを検出します。`model_c` を JIT レンダリングに切り替え、`model_c+` を JIT 静的解析に選択します。
- `model_a` と `model_b` は引き続き AOT コンパイルの対象となるため、Fusion は上記のイントロスペクションなしの例と同じように処理します。`model_d` は引き続き AOT レンダリングの対象となりますが、解析は対象外です。
- `model_b` が実行されると、Fusion は（更新されたばかりのデータを使用して）`model_c` の SQL をレンダリングし、解析して実行します。これら 3 つのステップはすべて連続して実行されます。
- `model_d` の AOT レンダリングされた SQL が解析され、実行されます。

<Expandable alt_header="Complex DAG with an introspective branch" is_open="true">
  <video src="/img/fusion/FusionJitRunUnsafeComplexDag.mp4" autoPlay loop muted style={{ width: "100%", maxWidth: 950 }} />
</Expandable>

ご想像のとおり、分岐型DAGはJITコンポーネントに進む前に可能な限りAOTコンパイルを行い、複数の `--threads` が利用可能な場合はそれらも実行します。ここでは、`model_b` の実行が完了するとすぐに `model_c` のレンダリングを開始できますが、AOTコンパイルされた `model_x` と `model_y` は別々に実行されます。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />
