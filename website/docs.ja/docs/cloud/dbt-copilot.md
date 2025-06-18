--- 
title: "About dbt Copilot" 
sidebar_label: "About dbt Copilot" 
description: "dbt Copilot is a powerful AI-powered assistant designed to accelerate your analytics workflows throughout your entire ADLC." 
pagination_next: "docs/cloud/enable-dbt-copilot"
keywords: ["dbt Copilot", "dbt", "AI", "AI-powered", "dbt"]
---

# About dbt Copilot <Lifecycle status="self_service,managed,managed_plus" /> 

<IntroText>
<Constant name="copilot" /> は、<Constant name="cloud" /> エクスペリエンスに完全に統合された強力な AI 搭載アシスタントで、分析ワークフローを加速するように設計されています。

</IntroText>

<Constant name="copilot" /> は、[アナリティクス開発ライフサイクル (ADLC)](https://www.getdbt.com/resources/guides/the-analytics-development-lifecycle) のあらゆる段階に AI を活用した支援を組み込み、豊富なメタデータを活用して関係性、系統、コンテキストをキャプチャすることで、洗練された信頼性の高いデータ製品を迅速に提供できるようにします。

<Constant name="copilot" /> では、自動コード生成と自然言語プロンプトを使用することで、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-copilot)、[<Constant name="visual_editor" /> (ベータ版)](/docs/cloud/build-canvas-copilot)、および [<Constant name="query_page" /> (ベータ版)](/docs/explore/dbt-insights) でボタンをクリックするだけで、[コードの生成](/docs/cloud/use-dbt-copilot)、[ドキュメント](/docs/build/documentation)、[テスト](/docs/build/data-tests)、[メトリック](/docs/build/metrics-overview)、[セマンティック モデル](/docs/build/semantic-models) を行うことができます。

:::tip
<Constant name="copilot" />は、Starter、Enterprise、Enterprise+アカウントでご利用いただけます。[デモを予約](https://www.getdbt.com/contact)して、AI駆動開発がワークフローをどのように効率化できるかをご確認ください。
:::

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/dbt-copilot-doc.gif" width="100%" title="Example of using dbt Copilot to generate documentation in the IDE" />

## dbt Copilot の仕組み

<Constant name="copilot" /> は、データのプライバシーとセキュリティを確保しながら、反復的なタスクを自動化することで効率を高めます。仕組みは以下のとおりです。

- <Constant name="copilot" /> へのアクセス方法:
- [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-copilot) を使用して、ドキュメント、テスト、セマンティックモデルを生成します。
- [<Constant name="visual_editor" /> (ベータ版)](/docs/cloud/build-canvas-copilot) を使用して、自然言語プロンプトから SQL コードを生成します。<Lifecycle status="managed,managed_plus" />
- [<Constant name="query_page" /> (ベータ版)](/docs/explore/dbt-insights) を使用して、自然言語プロンプトから分析用の SQL クエリを生成します。 <Lifecycle status="managed,managed_plus" />
- <Constant name="copilot" /> はメタデータ（列名、モデルSQL、ドキュメントなど）を収集しますが、行レベルのウェアハウスデータにはアクセスしません。
- メタデータとユーザープロンプトは、API呼び出しを通じてAIプロバイダー（この場合はOpenAI）に送信され、処理されます。
- AIによって生成されたコンテンツは <Constant name="cloud" /> に返され、プロジェクトファイル内で確認、編集、保存できます。
- <Constant name="copilot" /> は、ウェアハウスデータを使用してAIモデルをトレーニングしません。
- 使用状況データを除き、dbt Labsのシステムには機密データは保存されません。
- ユーザーがクエリに挿入した個人情報や機密データを含むクライアントデータは、OpenAIによって30日以内に削除されます。
- <Constant name="copilot" /> は、チーム間の一貫性を確保するためにベストプラクティスのスタイルガイドを使用しています。
- <Constant name="copilot" /> は、OpenAI の `gpt-3.x`、`gpt-4o`、`gpt-4.1-[mini|nano]`、および `gpt-4.5`（OpenAI により非推奨）モデル向けに最適化されています。`o1` や `o2` などの他のモデルはサポートされておらず、<Constant name="copilot"/> では動作しません。

:::tip
<Constant name="copilot" /> は分析エンジニアの作業を加速させますが、代替するものではありません。より優れたデータ製品をより早く提供するのに役立ちますが、AIが生成したコンテンツは不正確である可能性があるため、必ずレビューを行ってください。
:::
