---
title: dbt セットアップについて
id: about-setup
description: "About setup of dbt Core and Cloud"
sidebar_label: "About dbt setup"
pagination_next: "docs/environments-in-dbt"
pagination_prev: null
---

dbt は、データ プラットフォームに対して分析コードをコンパイルして実行し、お客様とチームがメトリクス、インサイト、ビジネス定義の唯一の真実のソースで共同作業できるようにします。dbt の導入には 2 つのオプションがあります。

**dbt Cloud** は、ブラウザベースのインターフェースを備えたホスト型 (シングル テナントまたはマルチ テナント) 環境で dbt Core を実行します。直感的なユーザー インターフェースは、さまざまなコンポーネントの設定に役立ちます。dbt Cloud には、ジョブのスケジュール設定、CI/CD、ドキュメントのホスティング、監視、アラートのターンキー サポートが備わっています。また、統合開発環境 (IDE) も提供しており、ローカル コマンド ライン (CLI) またはコード エディターから dbt コマンドを開発して実行できます。

**dbt Core** は、環境にローカルにインストールできるオープン ソース コマンド ライン ツールで、アダプターを介してデータベースとの通信が容易になります。

どのソリューションが適しているか分からない場合は、[dbt とは](/docs/introduction) および [dbt Cloud の機能](/docs/cloud/about-cloud/dbt-cloud-features) の記事を読んで判断してください。それでも質問がある場合は、[お問い合わせ](https://www.getdbt.com/contact/) までお気軽にお問い合わせください。

今すぐ dbt の構成を開始するには、適切なオプションを選択してください。

<div className="grid--2-col">

<Card
    title="dbt Cloud setup"
    body="Learn how to connect to a data platform, integrate with secure authentication methods, and configure a sync with a git repo."
    link="/docs/cloud/about-cloud-setup"
    icon="dbt-bit"/>

<Card
    title="dbt Core setup"
    body="Learn about dbt Core and how to setup data platform connections."
    link="/docs/core/about-core-setup"
    icon="dbt-bit"/>

</div>
