--- 
title: "Enable dbt Copilot" 
sidebar_label: "Enable dbt Copilot" 
description: "Enable dbt Copilot, an AI-powered assistant, in dbt to speed up your development." 
---

# dbt Copilot を有効にする<Lifecycle status="self_service,managed,managed_plus" /> 

<IntroText>
AI 搭載アシスタントである <Constant name="copilot" /> を <Constant name="cloud" /> で有効にすると、開発をスピードアップし、高品質なデータの提供に集中できます。
</IntroText>

このページでは、<Constant name="cloud" /> で <Constant name="copilot" /> を有効にして開発をスピードアップし、高品質なデータの提供に集中できるようにする方法について説明します。

## 前提条件

- <Constant name="cloud" /> でのみ利用可能です。
- [<Constant name="cloud" /> Starter、Enterprise、または Enterprise+ アカウント](https://www.getdbt.com/pricing)が必要です。
- [BYOK](#bringing-your-own-openai-api-key-byok)、[Canvas での自然なプロンプト](/docs/cloud/build-canvas-copilot)などの一部の機能は、Enterprise および Enterprise+ プランでのみ利用可能です。
- 継続的なアップデートを受け取るには、開発環境がサポートされている [リリーストラック](/docs/dbt-versions/cloud-release-tracks) である必要があります。
- デフォルトでは、<Constant name="copilot" /> のデプロイメントは、dbt Labs が管理する中央の OpenAI API キーを使用します。または、[独自の OpenAI API キーを提供](#bringing-your-own-openai-api-key-byok) することもできます。
- <Constant name="copilot" /> は、OpenAI の `gpt-3.x`、`gpt-4o`、`gpt-4.1-[mini|nano]`、および `gpt-4.5`（OpenAI により非推奨）モデル向けに最適化されています。`o1` や `o2` などの他のモデルはサポートされておらず、<Constant name="copilot"/> では動作しません。
- **アカウント設定** の次のセクションの手順に従って、AI 機能をオプトインしてください。

## dbt Copilot を有効にする

<Constant name="copilot" /> をオプトインするには、<Constant name="cloud" /> 管理者が以下の手順を実行できます。

1. ナビゲーションメニューの **アカウント設定** に移動します。
2. **設定** で、有効化するアカウントを確認します。
3. 右上の **編集** をクリックします。
4. **Copilot 機能へのアカウントアクセスを有効にする** オプションを有効にします。
5. **保存** をクリックします。これで、<Constant name="copilot" /> AI が有効になります。

注: 無効化するには（有効化後のみ）、手順 1 ～ 3 を繰り返し、手順 4 でオフに切り替え、手順 5 を繰り返します。

<Lightbox src="/img/docs/deploy/example-account-settings.png" width="90%" title="Example of the 'Enable account access to AI-powered feature' option in Account settings" />

## 独自の OpenAI API キーの持ち込み (BYOK) <Lifecycle status="managed_plus,managed" />

AI 機能が有効になったら、組織の OpenAI API キーを提供できます。<Constant name="cloud" /> は、OpenAI アカウントと利用規約を利用して <Constant name="copilot" /> を実行します。これにより、<Constant name="copilot" /> からのリクエストに対して、OpenAI から組織への課金が発生します。

AI キーを構成するには、以下を使用します。
- [dbt Labs が管理する OpenAI API キー](/docs/cloud/account-integrations?ai-integration=dbtlabs#ai-integrations)
- 独自の [OpenAI API キー](/docs/cloud/account-integrations?ai-integration=openai#ai-integrations)
- [Azure OpenAI](/docs/cloud/account-integrations?ai-integration=azure#ai-integrations)

構成の詳細については、[アカウント統合](/docs/cloud/account-integrations#ai-integrations) をご覧ください。
