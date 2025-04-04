---
title: "dbt と Unity Catalog のベストプラクティス"
id: "dbt-unity-catalog-best-practices"
description: Learn how to configure your.
displayText: Writing custom generic tests
hoverSnippet: Learn how to define your own custom generic tests.
---


Databricks dbt プロジェクトは、["Databricks dbt プロジェクトの設定方法ガイド"](/guides/set-up-your-databricks-dbt-project) に従って構成する必要があります。これで、Unity Catalog を使用して dbt プロジェクトの構築を開始する準備が整いました。ただし、まず、dbt ユーザーがさまざまなカタログと対話できるようにする方法を検討する必要があります。実稼働データの整合性を確保するために、次のベスト プラクティスをお勧めします:

## ブロンズ（ソース）データを分離する

Unity Catalog を使用することをお勧めします。これにより、他のカタログ、従来の Hive メタストア、外部メタストア、または Delta Live Table パイプライン出力から組織全体のデータを参照できるようになります。さらに、Databricks は [外部データと対話する](https://docs.databricks.com/external-data/index.html#interact-with-external-data-on-databricks) 機能を提供し、多くの [データベース ソリューション](https://docs.databricks.com/query-federation/index.html#what-is-query-federation-for-databricks-sql) へのクエリ フェデレーションをサポートしています。つまり、ソース データが別のカタログまたは外部データ ソースで定義されている場合でも、開発環境と運用環境からソース データにアクセスできます。

Bronze レイヤーの生データは、dbt [ソース](https://docs.getdbt.com/docs/build/sources) として定義し、開発と運用の両方ですべての dbt インタラクションに対して読み取り専用にする必要があります。デフォルトでは、すべての dbt 環境のすべての dbt ユーザーがこれらのすべての入力にアクセスできるようにすることをお勧めします。これにより、すべての環境での変換が同じ入力データから開始され、開発で観察された結果は、そのコードがデプロイされたときに複製されます。とはいえ、会社のデータ ガバナンス要件により、環境に応じて複数のワークスペースまたはデータ カタログを使用する必要がある場合があります。

環境に応じてソース データのデータ カタログ/スキーマが異なる場合は、[target.name](https://docs.getdbt.com/reference/dbt-jinja-functions/target#use-targetname-to-change-your-source-database) を使用して、環境に応じて取得するデータ カタログ/スキーマを変更できます。

複数の Databricks ワークスペースを使用して開発と運用を分離する場合は、接続構成文字列で dbt Cloud の [環境変数](https://docs.getdbt.com/docs/build/environment-variables) を使用して、1 つの dbt Cloud プロジェクトから複数のワークスペースを参照できます。SQL ウェアハウスでも同じことを実行して、環境に応じて異なるサイズにすることもできます。

これを行うには、接続設定で、Databricks ワークスペース URL のサーバー ホスト名と SQL ウェアハウスの HTTP パスに dbt の [環境変数構文](https://docs.getdbt.com/docs/dbt-cloud/using-dbt-cloud/cloud-environment-variables#special-environment-variables) を使用します。サーバー ホスト名は、検証チェックに合格するために有効なドメイン名である必要があるため、URL のドメイン サフィックス (例: `{{env_var('DBT_HOSTNAME')}}.cloud.databricks.com`) とウェアハウスのパス プレフィックス (例: `/sql/1.0/warehouses/{{env_var('DBT_HTTP_PATH')}}`) をハードコードする必要があります。

<Lightbox src="/img/guides/databricks-guides/databricks-connection-env-vars.png" title="Using environment variable syntax in connection configs" />

dbt Cloud で環境を作成するときに、環境変数を割り当てて接続情報を動的に入力できます。これらの環境の認証情報で使用するトークンが、関連付けられているワークスペースから生成されたものであることを確認してください。

<Lightbox src="/img/guides/databricks-guides/databricks-env-variables.png" title="Defining default environment variable values" />

## アクセス制御

データ コンシューマーにアクセス権を付与するには、dbt の [grants config](https://docs.getdbt.com/reference/resource-configs/grants) を使用して、dbt モデルによって生成されたデータベース オブジェクトに権限を適用します。これにより、すべての SQL を自分で記述するのではなく、構造化された辞書として権限を構成でき、dbt は最も効率的なパスを使用してそれらの権限を適用できます。

dbt を実行して消費者向けではないデータ ソースを読み取る権限については、以下の表にアクセス モデルをまとめています。実質的に、すべての開発者は、prod カタログの読み取り権限と dev カタログの書き込み権限のみを取得する必要があります。dbt を使用する場合、スキーマの作成は自動的に行われます。従来のデータ ウェアハウス ワークフローとは異なり、最上位のカタログ以外の Unity Catalog アセットを手動で作成する必要はありません。

**prod** サービス プリンシパルには、生のソース データに対する「読み取り」権限と、prod カタログに対する「書き込み」権限が必要です。**test** カタログと関連する dbt 環境を追加する場合は、専用のサービス プリンシパルを作成する必要があります。test サービス プリンシパルには、生のソース データに対する *読み取り* 権限と、**test** カタログに対する *書き込み* 権限が必要ですが、prod カタログまたは dev カタログに対する権限はありません。専用のテスト環境は、[CI テスト](https://www.getdbt.com/blog/adopting-ci-cd-with-dbt-cloud/) にのみ使用する必要があります。


**Table-level grants:**

|  | Source Data | Development catalog | Production catalog | Test catalog |
| --- | --- | --- | --- | --- |
| developers | select | select & modify | select or none | none |
| production service principal | select | none | select & modify | none |
| Test service principal | select | none | none | select & modify |


**Schema-level grants:**

|  | Source Data | Development catalog | Production catalog | Test catalog |
| --- | --- | --- | --- | --- |
| developers | use | use, create schema, table, & view | use or none | none |
| production service principal | use | none | use, create schema, table & view | none |
| Test service principal | use | none | none | use, create schema, table & view |


## 次のステップ

dbt を使用して Unity Catalog データセットの変換を開始する準備はできましたか?

ガイド、ヒント、ベスト プラクティスについては、以下のリソースをご覧ください:

- [dbt プロジェクトの構造化方法](/best-practices/how-we-structure/1-guide-overview)
- [自分のペースで学べる dbt 基礎トレーニング コース](https://learn.getdbt.com/courses/dbt-fundamentals)
- [CI/CD のカスタマイズ](/guides/custom-cicd-pipelines)
- [エラーのデバッグ](/guides/debug-errors)
- [カスタム汎用テストの作成](/best-practices/writing-custom-generic-tests)
- [dbt パッケージ ハブ](https://hub.getdbt.com/)
