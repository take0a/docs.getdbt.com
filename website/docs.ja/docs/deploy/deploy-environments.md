---
title: "Deployment environments"
id: "deploy-environments"
description: "Learn about dbt's deployment environment to seamlessly schedule jobs or enable CI."
---

<Constant name="cloud" /> のデプロイメント環境は、dbt ジョブを本番環境にデプロイし、dbt のメタデータや結果に依存する機能や統合を使用する上で不可欠です。dbt を実行するには、ジョブ実行中に使用される設定が環境によって決定されます。これには以下が含まれます。

- プロジェクトの実行に使用する <Constant name="core" /> のバージョン
- ウェアハウス接続情報（ターゲット データベース/スキーマ設定を含む）
- 実行するコードのバージョン

<Constant name="cloud" /> プロジェクトには複数のデプロイメント環境を含めることができるため、dbt ジョブの実行を柔軟かつカスタマイズして調整できます。デプロイメント環境を使用すると、[ジョブの作成とスケジュール設定](/docs/deploy/deploy-jobs#create-and-schedule-jobs)、[継続的インテグレーションの有効化](/docs/deploy/continuous-integration) など、特定のニーズや要件に基づいてさまざまな操作を実行できます。

:::tip <Constant name="cloud" /> 環境の管理方法を学ぶ
<Constant name="cloud" /> 環境を管理するためのさまざまなアプローチと、組織固有のニーズに合わせた推奨事項については、[<Constant name="cloud" /> 環境のベストプラクティス](/guides/set-up-ci)をご覧ください。
:::

開発環境とデプロイメント環境の詳細については、[<Constant name="cloud" /> 環境](/docs/dbt-cloud-environments) をご覧ください。

デプロイメント環境には次の 3 つのタイプがあります。
- **本番環境**: 本番環境で使用するデータの変換とパイプラインの構築を行う環境。
- **ステージング環境**: 本番環境データへのアクセスを制限しながら、本番環境ツールを使用する環境。
- **一般環境**: デプロイメント開発のための一般使用環境。

最終的な、信頼できるデプロイメントデータには、`Production` 環境タイプを使用することを強くお勧めします。最終的な本番環境ワークフロー用にマークできる環境は 1 つだけであり、この目的で `General` 環境を使用することはお勧めしません。

## デプロイメント環境を作成する

新しい <Constant name="cloud" /> デプロイメント環境を作成するには、「**デプロイ**」 -> 「**環境**」に移動し、「**環境の作成**」をクリックします。環境タイプとして「**デプロイメント**」を選択します。既に開発環境がある場合は、このオプションはグレー表示になります。

<Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/create-deploy-env.png" width="85%" title="Navigate to Deploy ->  Environments to create a deployment environment" />

### 本番環境として設定

<Constant name="cloud" /> では、各プロジェクトに1つのデプロイ環境（本番環境として機能します）を設定できます。本番環境は、<Constant name="explorer" /> やプロジェクト間参照などの機能を使用する上で不可欠です。<Constant name="cloud" /> におけるプロジェクトの本番環境の状態に関する信頼できる情報源として機能します。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/prod-settings-1.png" width="100%" title="Set your production environment as the default environment in your Environment Settings"/>

### セマンティックレイヤー

<Constant name="semantic_layer" /> をご利用のお客様の場合、環境設定の次のセクションは <Constant name="semantic_layer" /> 構成です。[<Constant name="semantic_layer" /> セットアップガイド](/docs/use-dbt-semantic-layer/setup-sl) に最新のセットアップ手順が記載されています。

また、dbt ジョブスケジューラを利用して [CI ジョブでセマンティックノードを検証](/docs/deploy/ci-jobs#semantic-validations-in-ci) し、dbt モデルへのコード変更がこれらのメトリックに違反していないことを確認することもできます。

## ステージング環境

ステージング環境を使用すると、開発者にデプロイメントワークフローとツールへのアクセスを許可しながら、本番環境データへのアクセスを制御できます。ステージング環境を使用すると、<Constant name="cloud" /> 内の単一プロジェクトの範囲内で、権限、データウェアハウス接続、データ分離をよりきめ細かく制御できます。

### Git ワークフロー

これにはいくつかのアプローチがありますが、最も簡単な方法は、プライマリブランチ（例：`main`）と類似しているものの独立した長期ブランチ（例：`staging`）をステージング環境に設定することです。

このシナリオでは、ワークフローは開発環境 -> ステージング環境 -> 本番環境へと上流へと移行し、開発者ブランチが `staging` ブランチにフィードされ、最終的に `main` にマージされるのが理想的です。多くの場合、マージ後、`main` ブランチと `staging` ブランチは同一になり、`development` ブランチからの次の一連の変更が反映されるまでそのまま残ります。`staging` にも `main` と同様のブランチ保護ルールを設定することをお勧めします。

お客様によっては、開発環境とステージング環境を `main` ブランチに接続し、リリースブランチを定期的に（毎日または毎週）作成して本番環境にフィードすることを好まれる場合があります。

### ステージング環境を使用する理由

ステージング環境を使用する主な理由は次のとおりです。
1. 変更を本番環境にデプロイする前の検証レイヤーとして追加できます。ステージング環境では、dbt モデルをデプロイ、テスト、および調査できます。
2. 開発ワークフローと本番環境データを明確に分離できます。これにより、開発者は本番環境のデプロイメントのデータにアクセスすることなく、延期やプロジェクト間参照などの機能を使用して、メタデータを活用した方法で作業できます。
3. 開発者がステージング環境でアドホックジョブを作成、編集、トリガーできるようにし、同時に本番環境を [環境レベルの権限](/docs/cloud/manage-access/environment-permissions) によってロックダウンできるようにします。

**ソースの条件付き構成** を使用すると、実行している環境に応じて、「prod」または「non-prod」のソース データを指すことができます。たとえば、このソースは `<DATABASE>.sensitive_source.table_with_pii` を指します。ここで、`<DATABASE>` は環境変数に基づいて動的に解決されます。

<File name="models/sources.yml">

```yaml
sources:
  - name: sensitive_source
    database: "{{ env_var('SENSITIVE_SOURCE_DATABASE') }}"
    tables:
      - name: table_with_pii
```

</File>

ソースは 1 つだけ（`sensitive_source`）あり、下流のすべての dbt モデルは `{{ source('sensitive_source', 'table_with_pii') }}` としてそこから選択します。プロジェクト内のコードと DAG の形状は、環境間で一貫性を保ちます。このように設定することで、ソースを重複させることなく、いくつかの重要なメリットが得られます。

**dbt Mesh におけるプロジェクト間参照:** モデルでプロジェクト間参照が構成されている `Project A` の下流に `Project B` があるとします。開発者が `Project B` の IDE で作業する場合、プロジェクト間参照は本番環境ではなく `Project A` のステージング環境に解決されます。ステージング環境でジョブを実行すると、これらの参照で同じ結果が得られます。本番環境のみが本番環境データを参照するため、別のプロジェクトを必要とせずにデータとアクセスが分離されます。

**遅延による開発の高速化:** `Project B` にもステージング環境のデプロイメントがある場合、`Project B` 内の未構築の上流モデルへの参照は、[遅延](/docs/cloud/about-cloud-develop-defer) を使用してステージング環境に解決され、本番環境のモデルには解決されません。これにより、環境の明確な分離を維持しながら、開発者の時間とウェアハウスのコストを削減できます。

最後に、ステージング環境は [<Constant name="explorer" />](/docs/explore/explore-projects) に独自のビューを持ち、本番環境とプレ本番環境のデータ全体を確認できるようになります。

<Lightbox src="/img/docs/collaborate/dbt-explorer/explore-staging-env.png" width="85%" title="Explore in a staging environment" />


### ステージング環境を作成する

<Constant name="cloud" /> で、「**デプロイ**」->「**環境**」に移動し、「**環境を作成**」をクリックします。環境タイプとして「**デプロイメント**」を選択します。既に開発環境がある場合は、このオプションはグレー表示になります。

<Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/create-staging-environment.png" width="85%" title="Create a staging environment" />


[デプロイ資格情報](#deployment-connection)に記載されている手順に従って、環境設定の残りの部分を完了してください。

データウェアハウスの資格情報は、専用ユーザーまたはサービスプリンシパルのものを使用することをお勧めします。

## Deployment connection

:::info ウェアハウス接続

ウェアハウス接続は、<Constant name="cloud" /> アカウントのアカウントレベルで作成および管理され、環境に割り当てられます。ウェアハウスの種類を変更するには、新しい環境を作成することをお勧めします。

各プロジェクトには、同じウェアハウスの種類を持つ複数の接続（Snowflake アカウント、Redshift ホスト、BigQuery プロジェクト、Databricks ホストなど）を設定できます。接続の詳細（データベース、スキーマなど）の一部は、<Constant name="cloud" /> 環境設定のこのセクションでオーバーライドできます。
:::

このセクションでは、ウェアハウスオブジェクトの構築時にdbtがターゲットとするウェアハウス内の正確な場所を指定します。このセクションは、ウェアハウスプロバイダーによって表示が多少異なります。

すべてのウェアハウスにおいて、[拡張属性](/docs/dbt-cloud-environments#extended-attributes)を使用して、不足している設定や非アクティブ（グレー表示）の設定を上書きできます。

<WHCode>


<div warehouse="Postgres">

Postgresを使用している場合、すべての値はプロジェクトの接続から推測されるため、このセクションは表示されません。これらの値をオーバーライドするには、[拡張属性](/docs/dbt-cloud-environments#extended-attributes)を使用してください。

</div>

<div warehouse="Redshift">

Redshift を使用している場合、すべての値はプロジェクトの接続から推測されるため、このセクションは表示されません。これらの値を上書きするには、[拡張属性](/docs/dbt-cloud-environments#extended-attributes) を使用してください。

</div>

<div warehouse="Snowflake">

<Lightbox src="/img/docs/collaborate/snowflake-deploy-env-deploy-connection.png" width="85%" title="Snowflake Deployment Connection Settings"/>

#### 編集可能なフィールド

- **ロール**: Snowflake ロール
- **データベース**: ターゲットデータベース
- **ウェアハウス**: Snowflake ウェアハウス

</div>

<div warehouse="Bigquery">

BigQueryを使用している場合、すべての値はプロジェクトの接続から推測されるため、このセクションは表示されません。これらの値をオーバーライドするには、[拡張属性](/docs/dbt-cloud-environments#extended-attributes)を使用してください。

</div>

<div warehouse="Spark">

Spark を使用している場合、すべての値はプロジェクトの接続から推測されるため、このセクションは表示されません。これらの値をオーバーライドするには、[拡張属性](/docs/dbt-cloud-environments#extended-attributes) を使用してください。

</div>

<div warehouse="Databricks">

<Lightbox src="/img/docs/collaborate/databricks-deploy-env-deploy-connection.png" width="85%" title="Databricks Deployment Connection Settings"/>

#### 編集可能なフィールド

- **カタログ** (オプション): [Unity Catalog 名前空間](/docs/core/connect-data-platform/databricks-setup)

</div>

</WHCode>


### デプロイメント認証情報

このセクションでは、ウェアハウスへの接続時に使用する認証情報を指定できます。認証方法は、ウェアハウスと <Constant name="cloud" /> 層によって異なる場合があります。

すべてのウェアハウスにおいて、[拡張属性](/docs/dbt-cloud-environments#extended-attributes) を使用して、不足している設定や非アクティブ（グレー表示）の設定を上書きしてください。認証情報に関しては、テキストボックスとログにシークレット値が表示されないように、拡張属性を[環境変数](/docs/build/environment-variables) (`password: '{{ env_var(''DBT_ENV_SECRET_PASSWORD'') }}'`) で囲むことを推奨します。

<WHCode>

<div warehouse="Postgres">

<Lightbox src="/img/docs/collaborate/postgres-deploy-env-deploy-credentials.png" width="85%" title="Postgres Deployment Credentials Settings"/>

#### 編集可能なフィールド

- **ユーザー名**: 使用するPostgresユーザー名（通常はサービスアカウント）
- **パスワード**: 指定されたユーザーのPostgresパスワード
- **スキーマ**: 対象のスキーマ

</div>

<div warehouse="Redshift">

<Lightbox src="/img/docs/collaborate/postgres-deploy-env-deploy-credentials.png" width="85%" title="Redshift Deployment Credentials Settings"/>

#### 編集可能なフィールド

- **ユーザー名**: 使用するRedshiftユーザー名（通常はサービスアカウント）
- **パスワード**: 一覧表示されているユーザーのRedshiftパスワード
- **スキーマ**: ターゲットスキーマ

</div>

<div warehouse="Snowflake">

<Lightbox src="/img/docs/collaborate/snowflake-deploy-env-deploy-credentials.png" width="85%" title="Snowflake Deployment Credentials Settings"/>

#### 編集可能なフィールド

- **認証方法**: dbt がウェアハウスに接続する方法を指定します。
  - 次のいずれか: [**ユーザー名とパスワード**、**キーペア**]
- **ユーザー名とパスワード** の場合:
  - **ユーザー名**: 使用するユーザー名 (通常はサービスアカウント)
  - **パスワード**: リストされているユーザーのパスワード
- **キーペア** の場合:
  - **ユーザー名**: 使用するユーザー名 (通常はサービスアカウント)
  - **秘密鍵**: 秘密 SSH 鍵の値 (オプション)
  - **秘密鍵のパスフレーズ**: 秘密 SSH 鍵のパスフレーズの値 (オプション、必要な場合のみ)
- **スキーマ**: この環境のターゲットスキーマ

</div>

<div warehouse="Bigquery">

<Lightbox src="/img/docs/collaborate/bigquery-deploy-env-deploy-credentials.png" width="85%" title="Bigquery Deployment Credentials Settings"/>

#### 編集可能なフィールド

- **データセット**: ターゲットデータセット

不足している設定や非アクティブ（グレー表示）の設定を上書きするには、[拡張属性](/docs/dbt-cloud-environments#extended-attributes)を使用します。認証情報の場合は、テキストボックスやログにシークレット値が表示されないように、拡張属性を[環境変数](/docs/build/environment-variables)（`password: '{{ env_var(''DBT_ENV_SECRET_PASSWORD'') }}'`）で囲むことを推奨します。

</div>

<div warehouse="Spark">

<Lightbox src="/img/docs/collaborate/spark-deploy-env-deploy-credentials.png" width="85%" title="Spark Deployment Credentials Settings"/>

#### 編集可能なフィールド

- **トークン**: アクセストークン
- **スキーマ**: ターゲットスキーマ
</div>

<div warehouse="Databricks">

<Lightbox src="/img/docs/collaborate/spark-deploy-env-deploy-credentials.png" width="85%" title="Databricks Deployment Credentials Settings"/>

#### 編集可能なフィールド

- **トークン**: アクセストークン
- **スキーマ**: ターゲットスキーマ

</div>

</WHCode>

## 環境を削除する

import DeleteEnvironment from '/snippets.ja/_delete-environment.md';

<DeleteEnvironment />

## 関連ドキュメント

- [<Constant name="cloud" /> 環境のベストプラクティス](/guides/set-up-ci)
- [ジョブのデプロイ](/docs/deploy/deploy-jobs)
- [CI ジョブ](/docs/deploy/continuous-integration)
- [<Constant name="cloud" /> 内のジョブまたは環境の削除](/faqs/Environments/delete-environment-job)

