---
title: "環境について"
id: "environments-in-dbt"
hide_table_of_contents: true
pagination_next: null
---

ソフトウェア エンジニアリングでは、エンジニアがソフトウェアのユーザーに影響を与えることなくコードを開発およびテストできるようにするために環境が使用されます。通常、dbt には 2 種類の環境があります。

- **デプロイメントまたは本番** (または _prod_) - エンド ユーザーが操作する環境を指します。

- **開発** (または _dev_) - エンジニアが作業する環境を指します。つまり、エンジニアは _development_ で新しいコードの作成とテストを反復的に行うことができます。これらの変更に自信が持てるようになったら、コードを _production_ にデプロイできます。

従来のソフトウェア エンジニアリングでは、異なる環境で完全に異なるアーキテクチャが使用されることがよくあります。たとえば、Web サイトの開発バージョンと本番バージョンでは、異なるサーバーとデータベースが使用されることがあります。<Term id="data-warehouse">データ ウェアハウス</Term> も、別々の環境を持つように設計できます。 _production_ 環境は、エンドユーザーがクエリを実行する (多くの場合、BI ツール経由) リレーション (スキーマ、テーブル、<Term id="view">ビュー</Term> など) を指します。

開発および本番環境でプロジェクトをビルドして実行する方法を dbt Cloud または dbt Core に指示するように環境を構成します。

<div className="grid--2-col">

<Card
    title="Environments in dbt Cloud"
    body="Seamlessly configure development and deployment environments in dbt Cloud to control how your project runs in both the dbt Cloud IDE, dbt Cloud CLI, and dbt jobs."
    link="/docs/dbt-cloud-environments"
    icon="dbt-bit"/>

<Card
    title="Environments in dbt Core"
    body="Setup and maintain separate deployment and development environments through the use of targets within a profile file"
    link="/docs/core/dbt-core-environments"
    icon="command-line"/>

</div> <br />

## 関連ドキュメント

- [dbt Cloud 環境のベスト プラクティス](/guides/set-up-ci)
- [デプロイメント環境](/docs/deploy/deploy-environments)
- [dbt Core のバージョンについて](/docs/dbt-versions/core)
- [dbt Cloud で環境変数を設定する](/docs/build/environment-variables#special-environment-variables)
- [jinja で環境変数を使用する](/reference/dbt-jinja-functions/env_var)
