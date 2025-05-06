---
title: "Project dependencies"
id: project-dependencies
sidebar_label: "Project dependencies"
description: "dbt プロジェクト全体で公開モデルを参照する"
pagination_next: null
keyword: dbt mesh, project dependencies, ref, cross project ref, project dependencies
---

# プロジェクトの依存関係 <Lifecycle status='enterprise'/>

dbt は長年にわたり、他のプロジェクトを [パッケージ](/docs/build/packages) としてインストールすることで、コードの再利用と拡張をサポートしてきました。他のプロジェクトをパッケージとしてインストールすると、そのプロジェクトの完全なソースコードが取り込まれ、自分のプロジェクトに追加されます。これにより、他のプロジェクトで定義されたマクロを呼び出したり、モデルを実行したりできるようになります。

これは、コードの再利用、ユーティリティマクロの共有、共通の変換の開始点を確立する上で優れた方法ですが、特に大規模な組織において、チーム間や大規模なコラボレーションを実現するには適していません。

dbt Labs は、複数の dbt プロジェクトにまたがる拡張された「依存関係」の概念をサポートしています。
- **パッケージ** &mdash; 使い慣れた既存の依存関係の種類。この依存関係を取得するには、パッケージの完全なソースコード（ソフトウェアライブラリなど）をインストールします。
- **プロジェクト** &mdash; 他のプロジェクトへの依存関係を取得する dbt メソッド。 dbt Cloud は、バックグラウンドで実行されるメタデータサービスを使用して、他のプロジェクトで定義された公開モデルへの参照をオンザフライで解決します。上流モデルを自分で解析したり実行したりする必要はありません。代わりに、これらのモデルへの依存関係を、データセットを返す API として扱います。公開モデルの品質と安定性を保証する責任は、そのモデルのメンテナーにあります。

## 前提条件
- [dbt Cloud Enterprise](https://www.getdbt.com/pricing) で利用可能です。使用するには、[パブリックモデル](/docs/collaborate/govern/model-access) を指定し、[プロジェクト間参照](#how-to-write-cross-project-ref) を追加してください。
- アップストリーム（「プロデューサー」）プロジェクトのセットアップ：
  - アップストリーム プロジェクトでモデルを [`access: public`](/reference/resource-configs/access) で設定し、`access` を定義した後に少なくとも 1 つのジョブが正常に実行されるようにします。
  - アップストリーム プロジェクトでデプロイメント環境を [本番環境](/docs/deploy/deploy-environments#set-as-production-environment) として定義し、その環境で少なくとも 1 つのジョブが正常に実行されるようにします。
  - アップストリーム プロジェクトにステージング環境がある場合は、そのステージング環境でジョブを実行し、ダウンストリームのプロジェクト間参照が解決されることを確認します。
- 各プロジェクト `name` は、dbt Cloud アカウント内で一意である必要があります。たとえば、`jaffle_marketing` チームの dbt プロジェクト（コードベース）がある場合、`Jaffle Marketing - Dev` と `Jaffle Marketing - Prod` のプロジェクトを作成しないでください。代わりに、[環境レベルの分離](/docs/dbt-cloud-environments#types-of-environments)を使用してください。
  - dbt Cloud は、[接続](/docs/cloud/connect-data-platform/about-connections#connection-management) をサポートしており、すべての dbt Cloud ユーザーが利用できます。接続により、環境ごとに異なるデータプラットフォーム接続が可能になり、プロジェクトを重複させる必要がなくなります。プロジェクトでは、同じウェアハウスタイプの複数の接続を使用できます。接続は、プロジェクトや環境間で再利用できます。
- `dbt_project.yml` ファイルでは大文字と小文字が区別されるため、プロジェクト名は `dependencies.yml` 内の名前と完全に一致する必要があります。たとえば、「JAFFLE_MARKETING」ではなく「jaffle_marketing」です。

import UseCaseInfo from '/snippets/_packages_or_dependencies.md';

<UseCaseInfo/>

## 例

例えば、Jaffle Shopのマーケティングチームに所属しているとします。チームのプロジェクト名は「jaffle_marketing」です。

<File name="dbt_project.yml">

```yml
name: jaffle_marketing
```

</File>

マーケティングデータのモデリングの一環として、他の2つのプロジェクトに依存する必要があります。
- `dbt_utils`（[パッケージ](#packages-use-case)）：独自のモデルのSQLを記述する際に使用できるユーティリティマクロのコレクションです。このパッケージはオープンソースで公開されており、dbt Labsによってメンテナンスされています。
- `jaffle_finance`（[プロジェクトユースケース](#projects-use-case)）：Jaffle Shopの収益に関するデータモデルです。このプロジェクトは非公開で、財務チームの同僚によってメンテナンスされています。このプロジェクトの最終モデルからいくつかを選択し、独自の作業の出発点とします。

<File name="dependencies.yml">

```yml
packages:
  - package: dbt-labs/dbt_utils
    version: 1.1.1

projects:
  - name: jaffle_finance  # case sensitive and matches the 'name' in the 'dbt_project.yml'
```

</File>

ここでは何が起こっているのでしょうか？

`dbt_utils` パッケージ - `dbt deps` を実行すると、dbt はこのパッケージの全内容（100 個以上のマクロ）をソースコードとしてプルダウンし、環境に追加します。その後、独自のプロジェクトで定義されたマクロを呼び出すのと同じように、パッケージ内の任意のマクロを呼び出すことができます。

`jaffle_finance` プロジェクト - これは新しいシナリオです。パッケージをインストールする場合とは異なり、`jaffle_finance` プロジェクト内のモデルはソースコードとしてプルダウンされ、プロジェクトに解析されることはありません。代わりに、dbt Cloud は、`jaffle_finance` プロジェクトで定義された [**公開モデル**](/docs/collaborate/govern/model-access) への参照を解決するメタデータ サービスを提供します。

### メリット

他のチームの作業をベースに構築する場合、この方法で参照を解決することにはいくつかのメリットがあります。
- モデルのメンテナーが `access: public` で指定した意図的なインターフェースを使用できます。
- プロジェクトのスコープを狭く保ち、不要なリソースや複雑さを回避できます。これにより、開発者と dbt の作業が高速化されます。
- `vars`、環境変数、`target.name` など、上流プロジェクトの条件付き設定をミラーリングする必要はありません。財務チームが本番環境でモデルをビルドしている場所であればどこでも、これらの設定を直接参照できます。財務チームがモデル名の変更、スキーマ名の変更、バージョンのアップグレードなどを行った場合でも、`ref` は正常に解決されます。
- `dbt run` や `dbt build` で誤ってモデルをビルドしてしまうリスクを排除できます。これらのモデルを選択することはできますが、実際に構築することはできません。これにより、予期せぬ倉庫コストや権限の問題を回避できます。また、各チームのモデルの適切な所有権とコスト配分も確保されます。

### プロジェクト間 ref の書き方

**`ref` の書き方:** `project` タイプの依存関係から参照されるモデルには、プロジェクト名を含む [2 つの引数を持つ `ref`](/reference/dbt-jinja-functions/ref#ref-project-specific-models) を使用する必要があります。

<File name="models/marts/roi_by_channel.sql">

```sql
with monthly_revenue as (
  
    select * from {{ ref('jaffle_finance', 'monthly_revenue') }}

),

...

```

</File>

#### サイクル検出

import CycleDetection from '/snippets/_mesh-cycle-detection.md';

<CycleDetection />

dbt Mesh の使用方法の詳細については、専用の [dbt Mesh ガイド](/best-practices/how-we-mesh/mesh-1-intro) と、無料で利用できる [dbt Mesh 学習コース](https://learn.getdbt.com/courses/dbt-mesh) を参照してください。

### ステージング環境による本番環境データの保護

開発環境で作業する場合、プロジェクト間の `ref` は通常、プロジェクトの本番環境に解決されます。ただし、本番環境データを保護するには、プロジェクト内に [ステージング デプロイメント環境](/docs/deploy/deploy-environments#staging-environment) を設定してください。

プロジェクトにステージング環境が統合されていると、コンシューマーもステージング環境にある場合、dbt Mesh はプロデューサーのステージング環境から公開モデル情報を自動的に取得します。同様に、コンシューマーが本番環境にある場合、dbt Mesh はプロデューサーの本番環境からモデル情報を取得します。これにより、環境間の一貫性が確保され、開発ワークフロー中に本番環境データへのアクセスを防ぐことでセキュリティが強化されます。

[ステージング環境を使用する理由](/docs/deploy/deploy-environments#why-use-a-staging-environment) で、そのメリットについて詳しくご覧ください。

#### 下流の依存関係を持つステージング環境

dbt Cloud は、プロジェクトにステージング環境が存在するとすぐに、本番環境へのフェイルオーバーなしに、下流のプロジェクトからのプロジェクト間参照を解決するためにステージング環境の使用を開始します。つまり、構成されたステージング環境で実行が成功していない場合でも、dbt Cloud は常にステージング環境のメタデータを使用して下流のプロジェクト内の参照を解決します。

下流の開発者のダウンタイムを回避するために、環境をステージングとしてマークする前に、ジョブを定義してトリガーする必要があります。

1. 新しい環境を作成しますが、**ステージング** としてマークしないでください。
2. その環境でジョブを定義します。
3. ジョブの実行をトリガーし、正常に完了することを確認します。
4. 環境を更新して、**ステージング** としてマークします。

### 比較

`jaffle_finance` プロジェクトを `package` 依存関係としてインストールした場合、完全なソースコードをダウンロードしてランタイム環境に追加することになります。これは、以下のことを意味します。
- dbt はより多くの入力を解析して解決する必要があり、処理速度が低下します。
- dbt は、これらのモデルを独自のモデルであるかのように設定する必要があります（`vars`、環境変数などを使用）。
- 明示的に `--exclude` を指定しない限り、dbt はこれらのモデルを独自のモデルとして実行します。
- プロジェクトのモデルを、メンテナー（財務チーム）が意図しない方法で使用してしまう可能性があります。

別の社内プロジェクトをパッケージとしてインストールすることが有効なパターンとなるケースがいくつかあります。
- 統合デプロイメント - 本番環境で、Jaffle Shop の中央データプラットフォームチームが `jaffle_finance` と `jaffle_marketing` の両方にモデルのデプロイメントをスケジュールする場合、dbt の [選択構文](/reference/node-selection/syntax) を使用して、両方のプロジェクトをパッケージとしてインストールする新しい「パススルー」プロジェクトを作成できます。
- 調整された変更 - 開発環境で、ステージング環境または本番環境に変更を導入する前に、上流プロジェクト (`jaffle_finance.monthly_revenue`) の公開モデルへの変更が下流モデル (`jaffle_marketing.roi_by_channel`) に与える影響をテストする場合、`jaffle_finance` パッケージを `jaffle_marketing` 内のパッケージとしてインストールできます。インストール時に特定の Git ブランチを指定することも可能ですが、両プロジェクトにまたがるエンドツーエンドのテストを頻繁に実行する必要がある場合は、これが安定したインターフェース境界を表しているかどうかを再検討することをお勧めします。

これは例外的なケースであり、一般的ではありません。他のチームのプロジェクトをパッケージとしてインストールすると、複雑さ、遅延、そして不要なコストが発生するリスクが増大します。チーム間で明確なインターフェース境界を定義し、あるチームの公開モデルを別のチームの「API」として提供し、開発者がより狭い範囲で開発できるようにすることで、より多くの人がより自信を持って貢献できるようになり、事前のコンテキスト設定も少なくて済みます。

## FAQs

<FAQ path="Project_ref/define-private-packages" />
<FAQ path="Project_ref/indirectly-reference-upstream-model" />

## 関連ドキュメント
- dbt Mesh の使用方法に関する詳しいガイダンスについては、[dbt Mesh](/best-practices/how-we-mesh/mesh-1-intro) ガイドをご覧ください。
- [dbt Mesh クイックスタート](/guides/mesh-qs)
