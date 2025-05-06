---
title: "dbtセマンティックレイヤーを設定する"
id: setup-sl
description: "直感的なナビゲーションを使用して、dbt Cloud で dbt セマンティック レイヤーをシームレスに設定します。"
sidebar_label: "Set up the Semantic Layer"
tags: [Semantic Layer]
pagination_next: "docs/use-dbt-semantic-layer/sl-architecture"
pagination_prev: "guides/sl-snowflake-qs"
---

dbt セマンティック レイヤーを使用すると、ビジネス メトリックを一元的に定義し、コードの重複と不整合を削減し、下流のツールでセルフサービスを作成するなどが可能になります。

## 前提条件

import SetUp from '/snippets/_v2-sl-prerequisites.md';

<SetUp/>

import SLCourses from '/snippets/_sl-course.md';

<SLCourses/>

## dbtセマンティックレイヤーを設定する

import SlSetUp from '/snippets/_new-sl-setup.md';  

<SlSetUp/>

<!--
1. Create a new environment in dbt Cloud by selecting **Deploy** and then **Environments**.
2. Select **dbt Version 1.6** (or the latest) and enter your deployment credentials.
3. To configure the new Semantic Layer, you must have a successful run in your new environment. We recommend running `dbt ls` since `dbt build` won’t succeed until you’ve created and defined semantic models and metrics.
4. To enable the dbt Semantic Layer, go to the **Account Settings** page and then select the specific project you want to enable the Semantic Layer for.
5. In the **Project Details** page, select **Configure Semantic Layer.** This will prompt you to enter data platform connection credentials for the Semantic Layer and select the environment where you want to enable the Semantic Layer. We recommend using a less privileged set of credentials when setting up your connection. The semantic layer requires SELECT and CREATE TABLE permissions.
6. After you’ve entered your credentials, you should see connection information that will allow you to connect to downstream tools. If the tool you are using can connect with JDBC, you can save the **JDBC URL** or each of the individual components provided (e.g., environment id, host). Alternatively, if the tool you connect to uses the Semantic Layer GraphQL API, save the GraphQL API host information.
7. Next, go back to the **Project Details** page and select **Generate Service Token** to create a Semantic Layer service token. Save this token for later.
8. You’re done 🎉! The semantic layer should is now enabled for your project. 
-->

## 次のステップ

- dbt セマンティック レイヤーの設定が完了したら、[利用可能な統合](/docs/cloud-integrations/avail-sl-integrations)を使用してメトリクスのクエリを開始します。
- 宣言型キャッシュを使用して、[クエリのパフォーマンスを最適化](/docs/use-dbt-semantic-layer/sl-cache)します。
- [CI でセマンティック ノードを検証](/docs/deploy/ci-jobs#semantic-validations-in-ci)し、dbt モデルへのコード変更によってこれらのメトリクスが損なわれないことを確認します。
- まだお試しでない場合は、お好みの開発ツールで[メトリクスとセマンティック モデルを構築する](/docs/build/build-metrics-intro)方法を学習してください。
- [dbt セマンティック レイヤーに関するよくある質問](/docs/use-dbt-semantic-layer/sl-faqs)をご確認ください。

## FAQs

<DetailsToggle alt_header="キャッシュはアクセス制御とどのように相互作用しますか?">

キャッシュされたデータは、基盤となるモデルとは別に保存されます。メトリクスがキャッシュから取得された場合、クエリ実行時にそれらのテーブルにセキュリティコンテキストが適用されません。

今後、認証情報を複製し、必要な最小限のアクセスレベルを特定し、それらの権限をキャッシュされたテーブルに適用する予定です。

</DetailsToggle>
