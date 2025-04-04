---
title: "CIジョブの最初のステップとして増分モデルをクローンする"
id: "clone-incremental-models"
description: Learn how to define clone incremental models as the first step of your CI job.
displayText: Clone incremental models as the first step of your CI job
hoverSnippet: Learn how to clone incremental models for CI jobs.
---

始める前に、いくつかの条件に注意する必要があります:
- `dbt clone` は、dbt バージョン 1.6 以降でのみ使用できます。dbt Cloud で新しいバージョンを有効にする方法については、[アップグレード ガイド](/docs/dbt-versions/upgrade-dbt-version-in-cloud) を参照してください。
- この戦略は、ゼロ コピー クローン作成をサポートするウェアハウスでのみ機能します (それ以外の場合、`dbt clone` はポインター ビューを作成するだけです)。
- チームによっては、増分モデルが増分モードとフル リフレッシュ モードの両方で実行されることをテストする必要がある場合があります。

dbt Cloud で [Slim CI ジョブ](/docs/deploy/continuous-integration) を作成し、次のように構成されているとします。

- 本番環境に従います。
- コマンド `dbt build --select state:modified+` を実行して、変更したすべてのモデルとその下流の依存関係を実行してテストします。
- チームの開発者がメイン ブランチに対して PR を開くたびにトリガーします。

<Lightbox src="/img/best-practices/slim-ci-job.png" width="70%" title="Example of a slim CI job with the above configurations" />

ここで、DAG 内の dbt プロジェクトが次のようになっていると想像してください:

<Lightbox src="/img/best-practices/dag-example.png" width="70%" title="Sample project DAG" />

`dim_wizards` を変更するプル リクエスト (PR) を開くと、CI ジョブが開始され、_変更されたモデルとその下流の依存関係_ (この場合は `dim_wizards` と `fct_orders`) のみが PR 固有の一時スキーマにビルドされます。

このビルドは、PR がメイン ブランチにマージされたときに発生する動作を模倣します。これにより、dbt プロジェクト全体をビルドする必要なく、重大な変更が発生しないことが保証されます。

## 変更されたモデルの 1 つ (またはその下流の依存関係の 1 つ) が増分モデルである場合はどうなりますか?

CI ジョブは変更されたモデルを PR 固有のスキーマに構築するため、`dbt build --select state:modified+` の最初の実行時に、変更された増分モデルは _PR 固有のスキーマにまだ存在しないため_ 完全に構築され、[is_incremental は false になります](/docs/build/incremental-models#understand-the-is_incremental-macro)。`full-refresh` モードで実行されています。

これは、次の理由で最適ではない可能性があります:
- 通常、増分モデルは最大のデータセットであるため、完全に構築するには長い時間がかかり、開発時間が遅くなり、ウェアハウス コストが高くなる可能性があります。
- CI ジョブで増分モデルの `full-refresh` が正常に完了しても、PR がメインにマージされると、prod での同じテーブルの _incremental_ ビルドが失敗する場合があります ([on_schema_change](/docs/build/incremental-models#what-if-the-columns-of-my-incremental-model-change) 構成が `fail` に設定されているスキーマ ドリフトを考えてください)。

これらの問題は、CI ジョブの最初のステップとして `dbt clone` コマンドを使用して、関連する既存の増分モデルを PR 固有のスキーマにゼロ コピー クローンすることで軽減できます。この方法では、`dbt build --select state:modified+` コマンドを最初に実行するときに、増分モデルが PR 固有のスキーマに既に存在するため、`is_incremental` フラグは `true` になります。

dbt Cloud CI チェックを実行するには、次の 2 つのコマンドが必要です:
1. 変更された、または変更された別のモデルの下流にある既存の増分モデルをすべて複製します:
  ```shell
  dbt clone --select state:modified+,config.materialized:incremental,state:old
  ```
1. 変更されたすべてのモデルとその下流の依存関係をビルドします:
  ```shell
  dbt build --select state:modified+
  ```

最初のクローン手順により、2 番目の手順の `dbt build` で選択された増分モデルは増分モードで実行されます。

<Lightbox src="/img/best-practices/clone-command.png" width="70%" title="Clone command in the CI config" />

CI ジョブの実行速度が上がり、PR がメインにマージされた後の動作をより正確に模倣できるようになります。

### 上記の [on_schema_change](/docs/build/incremental-models#what-if-the-columns-of-my-incremental-model-change) 構成が `fail` に設定されている場合に、「スキーマ ドリフトを考える」の拡張

次の構成の増分モデル `my_incremental_model` があるとします:

```sql

{{
    config(
        materialized='incremental',
        unique_key='unique_id',
        on_schema_change='fail'
    )
}}

```

ここで、`my_incremental_model` に新しい列を追加する PR を開いたとします。この場合:
- 増分ビルドは失敗します。
- `full-refresh` は成功します。

`--full-refresh` フラグなしで `dbt build` を実行するだけの毎日の本番ジョブがある場合、PR がメインにマージされてジョブが開始されると、失敗します。そこで、CI で何が起こるようにしたいですか?
- この PR がメインにマージされたら、本番環境で失敗を回避するために、本番環境ですぐに `dbt build --full-refresh --select my_incremental_model` を実行する必要があることがわかるように、CI でも失敗させたいですか? これにより、CI チェックが成功しなくなります。
- 本番環境でこのモデルに対して `full-refresh` を実行すると、成功した状態になるため、CI チェックを成功させたいですか?この PR をメインにマージしたときに本番ジョブが突然失敗し、本番環境で `dbt build --full-refresh --select my_incremental_model` を実行する必要があることを覚えていない場合、これは不愉快な驚きにつながる可能性があります。

おそらく、ここには完璧な解決策はありません。すべてはトレードオフです。私たちが好むのは、失敗した CI ジョブを用意し、ブロッキング ブランチ保護ルールを手動でオーバーライドして、予期せぬ事態が起こらないようにし、PR がマージされたらプロダクションで適切なコマンドを積極的に実行できるようにすることです。

### 「なぜ `state:old` なのか」の拡張

まったく新しい増分モデルの場合、CI では `full-refresh` モードで実行する必要があります。これは、PR が `main` にマージされると、本番環境で `full-refresh` モードで実行されるためです。また、本番環境にはまだ存在していません...まったく新しいモデルです!
これを指定しないと、エラーは表示されず、「状態マニフェストに関係が見つかりません...」というメッセージが表示されます。したがって、技術的には `state:old` を指定しなくても機能しますが、`state:old` を追加するとより明示的になり、まったく新しい増分モデルのクローン作成も試行されなくなります。
