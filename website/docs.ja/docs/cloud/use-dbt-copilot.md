--- 
title: "Use dbt Copilot" 
sidebar_label: "Use dbt Copilot" 
description: "Use dbt Copilot to generate documentation, tests, semantic models, and sql code from scratch, giving you the flexibility to modify or fix generated code." 
---

import CopilotResources from '/snippets.ja/_use-copilot-resources.md';
import CopilotEditCode from '/snippets.ja/_use-copilot-edit-code.md';
import CopilotVE from '/snippets.ja/_use-copilot-ve.md';

# Use dbt Copilot <Lifecycle status="self_service,managed,managed_plus" /> 

<IntroText>
<Constant name="copilot" /> を使用すると、ドキュメント、テスト、セマンティック モデル、コードを最初から生成できるため、生成されたコードを柔軟に変更または修正できます。

</IntroText>

このページでは、<Constant name="copilot" /> を使用して次の操作を行う方法について説明します。

- [リソースの生成](#generate-resources) - [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) での開発中に、<Constant name="copilot" /> の生成ボタンを使用してドキュメント、テスト、セマンティック モデル ファイルを生成することで時間を節約できます。
- [SQL をインラインで生成および編集](#generate-and-edit-sql-inline) - 自然言語プロンプトを使用して SQL コードを最初から生成したり、キーボード ショートカットを使用したり、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) でコードをハイライト表示したりすることで、既存の SQL ファイルを編集したりできます。
- [ビジュアル モデルの構築](#build-visual-models) - <Constant name="copilot" /> を使用して、[<Constant name="visual_editor" />](/docs/cloud/use-canvas) で自然言語プロンプトを使用してモデルを生成します。
- [クエリの作成](#build-queries) &mdash; <Constant name="copilot" /> を使用して、[<Constant name="query_page" />](/docs/explore/dbt-insights) で自然言語プロンプトを使用した探索的データ分析用のクエリを生成します。

## リソースを生成する

<CopilotResources/>

## SQLをインラインで生成および編集する

<CopilotEditCode/>

<Constant name="copilot" /> は [<Constant name="visual_editor" />](/docs/cloud/canvas) とシームレスに統合されます。これは、自然言語プロンプトを使用してビジュアルモデルを構築できるドラッグアンドドロップエクスペリエンスです。開始する前に、[<Constant name="visual_editor" />](/docs/cloud/use-canvas#access-canvas) にアクセスできることを確認してください。

<CopilotVE/>

<Constant name="copilot" /> を使用して [<Constant name="query_page" />](/docs/explore/dbt-insights) で自然言語プロンプトを使ったクエリを作成し、直感的でコンテキスト豊富なインターフェースでシームレスにデータを探索およびクエリできます。開始する前に、[<Constant name="query_page" />](/docs/explore/access-dbt-insights) にアクセスできることを確認してください。

<Constant name="query_page" /> で自然言語プロンプトを使った SQL クエリの作成を開始するには、次の手順を実行します。

1. クエリコンソールのサイドバーメニューで、**<Constant name="copilot" />** アイコンをクリックします。
2. dbt Copilot プロンプトボックスに自然言語でプロンプトを入力すると、dbt Copilot によって必要な SQL クエリが作成されます。 <!--You can also reference existing models using the `@` symbol. For example, to build a model that calculates the total price of orders, you can enter `@orders` in the prompt and it'll pull in and reference the `orders` model.-->
3. **[送信]** をクリックすると、<Constant name="copilot" /> によって、作成する SQL クエリの概要が生成されます。プロンプトをクリアするには、**[クリア]** ボタンをクリックします。プロンプトボックスを閉じるには、<Constant name="copilot" /> アイコンをもう一度クリックします。
4. <Constant name="copilot" /> によって、クエリの説明を含む SQL が自動的に生成されます。
- 生成された SQL を既存のクエリに追加するには、**[追加]** をクリックします。
- 生成された SQL で既存のクエリを置き換えるには、**[置換]** をクリックします。
5. **クエリコンソールメニュー** で、**[実行]** ボタンをクリックしてデータをプレビューします。
6. 結果を確認するか、モデルの構築を続行します。

<Lightbox src="/img/docs/dbt-insights/insights-copilot.gif" width="95%" title="dbt Copilot in dbt Insights" />
