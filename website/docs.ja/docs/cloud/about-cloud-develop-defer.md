---
title: Using defer in dbt
id: about-cloud-develop-defer
description: "Learn how to leverage defer to prod when developing with dbt."
sidebar_label: "Defer in dbt"
pagination_next: "docs/cloud/cloud-cli-installation"
---


[Defer](/reference/node-selection/defer) は、開発者が編集したモデルのみをビルド、実行、テストできる強力な機能です。この機能により、開発者は、それ以前のすべてのモデル（上流の親）を最初に実行してビルドする必要はありません。dbt は、比較のために本番環境のマニフェストを使用することでこの機能を実現し、上流の本番環境の成果物を使用して `{{ ref() }}` 関数を解決します。

<Constant name="cloud_ide" /> と <Constant name="cloud" /> CLI の両方を使用することで、ユーザーは開発ワークフロー内で本番環境のメタデータをネイティブに直接参照できます。

<Lightbox src src="/img/docs/reference/defer-diagram.png" width="50%" title="Use 'defer' to modify end-of-pipeline models by pointing to production models, instead of running everything upstream." />

`--defer` を使用する場合、<Constant name="cloud" /> は `{{ ref() }}` 関数を解決するために、以下の実行順序に従います。

1. 遅延リレーションの開発バージョンが存在する場合、dbt は参照を解決する際に開発データベースの場所を優先的に使用します。
2. 開発バージョンが存在しない場合、dbt はステージング環境のメタデータに基づいて、親リレーションのステージング場所を使用します。
3. 開発バージョンもステージング環境も存在しない場合、dbt は本番環境のメタデータに基づいて、親リレーションの本番場所を使用します。dbt は、呼び出しごとにステージング環境または本番環境のいずれか 1 つの環境のみを遅延することに注意してください。

**注:** `--favor-state` フラグを渡すと、ステージングメタデータが利用可能な場合は常にそれを使用して参照を解決します。そうでない場合は、開発リレーションの有無に関係なく、デフォルトで本番環境のメタデータが使用され、手順 1 はスキップされます。

開発サイクルの開始時と終了時に開発スキーマを削除することをお勧めします。

本番環境データに対する追加の制御が必要な場合は、[ステージング環境](/docs/deploy/deploy-environments#staging-environment)を作成してください。dbt は、本番環境ではなくステージング環境を使用して `{{ ref() }}` 関数を解決します。

## 必要な設定

- **環境設定** ページで **[本番環境](/docs/deploy/deploy-environments#set-as-production-environment)** チェックボックスをオンにする必要があります。
  - これは、<Constant name="cloud" /> プロジェクトごとに 1 つのデプロイメント環境に対して設定できます。
- 事前にジョブを正常に実行しておく必要があります。

defer を使用する場合、CI ジョブを除く、直近の成功した本番ジョブの成果物と比較されます。

### dbt IDE での遅延

<Constant name="cloud_ide" /> で遅延を使用するには、デプロイジョブによって生成された本番環境のアーティファクトが必要です。<Constant name="cloud" /> は、まずステージング環境（存在する場合）で、そうでない場合は本番環境でこれらのアーティファクトの有無を確認します。

ステージング環境は存在するものの、デプロイジョブが実行されていない場合、<Constant name="cloud_ide" /> の遅延機能は機能しません。これは、ステージング環境でデプロイジョブが正常に実行されるまで、遅延に必要なメタデータが存在しないためです。

<Constant name="cloud_ide" /> で遅延を有効にするには、コマンドバーの「**ステージング/本番環境に遅延**」ボタンを切り替えます。有効にすると、<Constant name="cloud" /> は次の処理を実行します。

1. ステージング環境または本番環境から最新のマニフェストを取得し、比較します。
2. コマンドに `--defer` フラグを渡します（このフラグを受け入れるコマンドの場合）。

例えば、[開発スキーマに何もない](/reference/node-selection/defer#usage) 状態で新しいブランチで開発を開始し、1つのモデルを編集して `dbt build -s state:modified` を実行すると、編集したモデルのみが実行されます。`{{ ref() }}` 関数はすべて、参照先のモデルのステージング環境または本番環境の場所を参照します。

<Lightbox src="/img/docs/dbt-cloud/defer-toggle.jpg" width="100%" title="Select the 'Defer to production' toggle on the bottom right of the command bar to enable defer in the Studio IDE."/>

### Defer in dbt CLI

<Constant name="cloud_cli" /> と <Constant name="cloud_ide" /> で `--defer` を使用する場合の主な違いは、本番環境のアーティファクトとは異なり、<Constant name="cloud_cli" /> ではすべての呼び出しに対して `--defer` が*自動的に*有効になることです。`--no-defer` フラグを使用して無効にすることもできます。

<Constant name="cloud_cli" /> では、遅延アーティファクトのソース環境を選択できるため、柔軟性が向上します。`dbt_project.yml` ファイルまたは `dbt_cloud.yml` ファイルのいずれかで `defer-env-id` キーを手動で設定できます。デフォルトでは、<Constant name="cloud_cli" /> はプロジェクトの「ステージング」環境（定義されている場合）のメタデータを優先し、そうでない場合は「本番環境」のメタデータを優先します。

<File name="dbt_cloud.yml">

```yml
context:
  active-host: ...
  active-project: ...
  defer-env-id: '123456'
```

</File>


<File name="dbt_project.yml"> 

```yml
dbt-cloud:
  defer-env-id: '123456'
```

</File>
