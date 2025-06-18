--- 
title: "Copilot style guide" 
sidebar_label: Copilot style guide 
id: copilot-styleguide 
description: "Use the Copilot `dbt-styleguide.md` file for best practices and naming conventions in dbt projects." 
---

<IntroText>
このガイドでは、<Constant name="copilot" /> `dbt-styleguide.md` ファイルの概要を示し、その構造、推奨される使用法、および dbt プロジェクトで効果的に実装するためのベスト プラクティスについて説明します。
</IntroText>

`dbt-styleguide.md` は、dbt プロジェクトのスタイルガイドを作成するためのテンプレートです。以下の内容が含まれています。

- SQL スタイルガイドライン（例：小文字のキーワードや末尾のカンマの使用）
- モデルの構成と命名規則
- モデルの設定とテストの実践
- スタイルルールを適用するためのコミット前フックの使用に関する推奨事項

このガイドは、dbt プロジェクトの一貫性と明確性を確保するのに役立ちます。

## Copilot 用の `dbt-styleguide.md`

<Constant name="cloud_ide" /> で <Constant name="copilot" /> を使用すると、`dbt-styleguide.md` というスタイルガイドテンプレートを自動的に生成できます。スタイルガイドを手動で追加または編集する場合も、この命名規則に従う必要があります。<Constant name="copilot" /> では、他のファイル名は使用できません。

`dbt-styleguide.md` ファイルをプロジェクトのルートに追加します。<Constant name="copilot" /> は、[テスト](/docs/build/data-tests)、[メトリクス](/docs/build/metrics-overview)、[セマンティックモデル](/docs/build/semantic-models)、[ドキュメント](/docs/build/documentation) を生成する際に、このファイルを大規模言語モデル (LLM) のコンテキストとして使用します。

注意: <Constant name="copilot" /> の `​​dbt-styleguide.md` を作成すると、dbt のデフォルトのスタイル ガイドが上書きされます。

## Studio IDE で `dbt-styleguide.md` を作成する

1. <Constant name="cloud_ide" /> でファイルを開きます。
2. ツールバーの **<Constant name="copilot" />** をクリックします。
3. メニューから **Generate ... Style guide** を選択します。
<Lightbox src="/img/docs/dbt-cloud/generate-styleguide.png" title="Generate styleguide in Copilot" /> 
4. スタイルガイドテンプレートが <Constant name="cloud_ide" /> に表示されます。**[保存]** をクリックします。
`dbt-styleguide.md` がプロジェクトのルートレベルに追加されます。

以前にスタイルガイドファイルを生成したことがない場合は、最新バージョンが <Constant name="dbt_platform" /> から自動的に取得されます。

## `dbt-styleguide.md` が既に存在する場合

既存の `dbt-styleguide.md` ファイルがあり、新しいスタイルガイドを生成しようとすると、以下のオプションを含むモーダルが表示されます。

- **キャンセル** &mdash; 変更を加えずに終了します。
- **復元** &mdash; <Constant name="dbt_platform" /> の最新バージョンに戻します。
- **編集** &mdash; 既存のスタイルガイドを手動で変更します。

<Lightbox src="/img/docs/dbt-cloud/styleguide-exists.png" title="Styleguide exists" />

## さらに詳しく

- [dbt Copilot について](/docs/cloud/dbt-copilot)
- [dbt プロジェクトのスタイル設定方法](/best-practices/how-we-style/0-how-we-style-our-dbt-projects)