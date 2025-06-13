---
title: "Lint and format your code"
id: "lint-format"
description: Integrate with popular linters and formatters like SQL Fluff, sqlfmt, Black, and Prettier."
sidebar_label: "Lint and format"
tags: [IDE]
---

[SQLFluff](https://sqlfluff.com/)、[sqlfmt](http://sqlfmt.com/)、[Black](https://black.readthedocs.io/en/latest/)、[Prettier](https://prettier.io/)といった人気のリンターやフォーマッタと統合することで、開発ワークフローを強化します。これらの強力なツールは、開発フローを中断することなく、<Constant name="cloud_ide" /> 内で直接活用できます。

<details>
<summary>リンターとフォーマッタとは何ですか? </summary>
リンターはコードのエラー、バグ、スタイルの問題を分析し、フォーマッタはスタイルとフォーマットのルールを修正します。リンターとフォーマッタの使い分けについては、<a href="#faqs">FAQ</a>をご覧ください。
</details>


<Constant name="cloud_ide" /> では、5 種類のファイルタイプに対して lint チェック、自動修正、フォーマットを実行できます。

- SQL - SQLFluff による [Lint](#lint) と修正、sqlfmt による [format](#format)
- YAML、Markdown、JSON - Prettier によるフォーマット
- Python - Black によるフォーマット

ファイルタイプごとに独自の lint チェックおよびフォーマットルールがあります。lint チェックプロセスを [カスタマイズ](#customize-linting) することで、柔軟性を高め、問題やスタイルの検出を強化できます。

IDE はデフォルトで sqlfmt ルールを使用してコードをフォーマットするため、すぐに使用できます。ただし、dbt プロジェクトのルートディレクトリに `.sqlfluff` というファイルがある場合は、IDE はデフォルトで SQLFluff ルールを使用します。

<DocCarousel slidesPerView={1}>

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/sqlfluff.gif" width="100%" title="Use SQLFluff to lint/format your SQL code, and view code errors in the Code Quality tab."/>

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/sqlfmt.gif" width="95%" title="Use sqlfmt to format your SQL code."/>

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/prettier.gif" width="95%" title="Format YAML, Markdown, and JSON files using Prettier."/>

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-sql-popup.jpg" width="95%" title="Use the Config button to select your tool."/>

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-sqlfluff-config.jpg" width="95%" title="Customize linting by configuring your own linting code rules, including dbtonic linting/styling."/>

</DocCarousel>

## Lint

<Constant name="cloud_ide" /> を使用すると、設定可能な SQL リンターである [SQLFluff](https://sqlfluff.com/) をシームレスに使用して、複雑な関数、構文、フォーマット、コンパイルエラーを警告できます。この統合により、Cloud <Constant name="cloud_ide" /> 内で直接コードエラーのチェック、修正、表示を実行できます。

- Jinja および SQL と連携します。
- 組み込みの [リンティングルール](https://docs.sqlfluff.com/en/stable/rules.html) が付属しています。独自のリンティングルールを [カスタマイズ](#customize-linting) することもできます。
- **Lint** (リンティングエラーを表示し、対処方法を推奨) や **Fix** (<Constant name="cloud_ide" /> 内のエラーを自動修正) などのオプションを使用して、[リンティングを有効化](#enable-linting) できます。
- コード エラーを表示するための **コード品質** タブを表示し、コード品質の可視性と管理を提供し、使用されている SQLFluff のバージョンを表示します。

:::info エフェメラルモデルはサポートされていません
dbt v1.5以前のバージョンでは、Lintはエフェメラルモデルをサポートしていません。詳しくは[FAQ](#faqs)をご覧ください。
:::

### リンティングを有効にする

リンティングは、保護されたプライマリ Git ブランチを含むすべてのブランチで利用できます。<Constant name="cloud_ide" /> により保護されたブランチへのコミットがブロックされるため、新しいブランチに変更をコミットするように促されます。

1. リンティングを有効にするには、`.sql` ファイルを開き、[**コード品質**] タブをクリックします。
2. [コンソールセクション](/docs/cloud/dbt-cloud-ide/ide-user-interface#console-section) の右下、**ファイルエディタ** の下にある **`</> Config`** ボタンをクリックします。
3. コード品質ツールの設定ポップアップで、**sqlfluff** または **sqlfmt** を選択できます。
4. コードをリンティングするには、[**sqlfluff**] ラジオボタンを選択します。 (sqlfmt を使用してコードを[フォーマット](#format)します)
5. **sqlfluff** ラジオボタンを選択したら、コンソールセクション（**ファイルエディタ** の下）に戻り、**Lint** または **Fix** ドロップダウンボタンを選択します。
    - **Lint** ボタン &mdash; <Constant name="cloud_ide" /> 内のリンティングの問題が、**ファイルエディタ** に波線の下線で表示されます。下線付きの問題にマウスポインターを合わせると、詳細とアクションが表示されます。これには、すべての問題または特定の問題を修正するための **Quick Fix** オプションも含まれます。リンティング後、結果を確認するメッセージが表示されます。保存後にリンティングは再実行されません。リンティングを再実行するには、もう一度 [**Lint**] をクリックします。
    - **Fix** ボタン &mdash; **ファイルエディタ** 内のリンティングエラーを自動的に修正します。修正が完了すると、結果を確認するメッセージが表示されます。
    - **コード品質** タブを使用して、コード エラーを表示およびデバッグします。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-lint-format-console.gif" width="90%" title="Use the Lint or Fix button in the console section to lint or auto-fix your code."/>

### リンティングのカスタマイズ

SQLFluff は設定可能な SQL リンターです。IDE のデフォルトのリンティング設定を使用する代わりに、独自のリンティングルールを設定できます。標準の `.sqlfluffignore` ファイルを使用して、ファイルとディレクトリを除外できます。構文の詳細については、[.sqlfluffignore 構文ドキュメント](https://docs.sqlfluff.com/en/stable/configuration.html#id2) を参照してください。

独自のリンティングルールを設定するには:

1. ルートプロジェクトディレクトリ（ファイルの親ディレクトリまたは最上位ディレクトリ）に新しいファイルを作成します。注: ルートプロジェクトディレクトリは、`dbt_project.yml` ファイルが存在するディレクトリです。
2. ファイルに `.sqlfluff` という名前を付けます（`sqlfluff` の前に `.` を追加することを忘れないでください）。
3. [作成](https://docs.sqlfluff.com/en/stable/configuration/setting_configuration.html#new-project-configuration) し、カスタム構成コードを追加します。
4. 変更を保存してコミットします。
5. <Constant name="cloud_ide" /> を再起動します。
6. テストして、リンティングを楽しんでください！

#### スナップショットの lint 処理

デフォルトでは、<Constant name="cloud" /> はプロジェクト内の変更されたすべての `.sql` ファイル（スナップショットを含む）を lint します。[スナップショット](/docs/build/snapshots) は YAML および `.sql` ファイルで定義できますが、その SQL は lint 可能ではないため、lint 処理中にエラーが発生する可能性があります。

SQLFluff がスナップショットファイルを lint 処理しないようにするには、`.sqlfluffignore` ファイルにスナップショットディレクトリ（例：`snapshots/`）を追加してください。

<Constant name="cloud" /> はバックエンドのスナップショットを自動的に無視しないため、`.sqlfluffignore` ファイルでスナップショットを明示的に除外する必要があることに注意してください。

### dbtonic の linting ルールを設定する

私たちのプロジェクトで使用している dbt 固有の（または dbtonic の） linting ルールについては、[Jaffle shop SQLFluff 設定ファイル](https://github.com/dbt-labs/jaffle-shop-template/blob/main/.sqlfluff) を参照してください。

<details>
<summary>dbt Labs 提供の dbtonic 設定コード例</summary>

```
[sqlfluff]
templater = dbt
# This change (from jinja to dbt templater) will make linting slower
# because linting will first compile dbt code into data warehouse code.
runaway_limit = 10
max_line_length = 80
indent_unit = space

[sqlfluff:indentation]
tab_space_size = 4

[sqlfluff:layout:type:comma]
spacing_before = touch
line_position = trailing

[sqlfluff:rules:capitalisation.keywords] 
capitalisation_policy = lower

[sqlfluff:rules:aliasing.table]
aliasing = explicit

[sqlfluff:rules:aliasing.column]
aliasing = explicit

[sqlfluff:rules:aliasing.expression]
allow_scalar = False

[sqlfluff:rules:capitalisation.identifiers]
extended_capitalisation_policy = lower

[sqlfluff:rules:capitalisation.functions]
capitalisation_policy = lower

[sqlfluff:rules:capitalisation.literals]
capitalisation_policy = lower

[sqlfluff:rules:ambiguous.column_references]  # Number in group by
group_by_and_order_by_style = implicit
```
</details>

スタイルのベスト プラクティスの詳細については、[SQL のスタイル設定方法](/best-practices/how-we-style/2-how-we-style-our-sql) を参照してください。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/ide-sqlfluff-config.jpg" width="90%" title="Customize linting by configuring your own linting code rules, including dbtonic linting/styling."/>

## フォーマット

<Constant name="cloud_ide" /> では、ボタンをクリックするだけで、スタイルガイドに沿ってコードをフォーマットできます。<Constant name="cloud_ide" /> は、sqlfmt、Prettier、Black などのフォーマッタと統合されており、SQL、YAML、Markdown、Python、JSON の 5 種類のファイル形式のコードを自動的にフォーマットします。

- SQL - [sqlfmt](http://sqlfmt.com/) を使用してフォーマットします。これは、dbt SQL と Jinja をフォーマットする 1 つの方法です。
- YAML、Markdown、JSON - [Prettier](https://prettier.io/) を使用してフォーマットします。
- Python - [Black](https://black.readthedocs.io/en/latest/) を使用してフォーマットします。

Cloud <Constant name="cloud_ide" /> フォーマット統合により、コードのフォーマットなどの手動タスクが処理され、高品質なデータ モデルの作成、共同作業、影響力のある結果の実現に集中できるようになります。

### SQL のフォーマット

SQL コードをフォーマットするために、<Constant name="cloud" /> は [sqlfmt](http://sqlfmt.com/) と統合されています。sqlfmt は、SQL クエリと Jinja をフォーマットするための単一の方法を提供する、妥協のない SQL クエリフォーマッタです。

デフォルトでは、<Constant name="cloud_ide" /> は sqlfmt ルールを使用してコードをフォーマットするため、「**フォーマット**」ボタンがすぐに使用可能になり便利です。ただし、dbt プロジェクトのルートディレクトリに .sqlfluff というファイルがある場合、<Constant name="cloud_ide" /> はデフォルトで SQLFluff ルールを使用します。

フォーマットは、保護されたプライマリ Git ブランチを含むすべてのブランチで利用できます。<Constant name="cloud_ide" /> は保護されたブランチへのコミットをブロックするため、新しいブランチに変更をコミットするように求められます。

1. `.sql` ファイルを開き、「**コード品質**」タブをクリックします。
2. コンソールの右側にある **`</> Config`** ボタンをクリックします。
3. コード品質ツールの設定ポップアップで、sqlfluff または sqlfmt を選択できます。
4. コードをフォーマットするには、**sqlfmt** ラジオボタンを選択します。(sqlfluff を使用してコードの [lint](#linting) を実行します)。
5. **sqlfmt** ラジオボタンを選択したら、コンソールセクション (**ファイルエディタ** の下) に移動して、**フォーマット** ボタンを選択します。
6. **フォーマット** ボタンをクリックすると、**ファイルエディタ** 内のコードが自動フォーマットされます。自動フォーマットが完了すると、結果を確認するメッセージが表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/sqlfmt.gif" width="90%" title="Use sqlfmt to format your SQL code."/>

### YAML、Markdown、JSON のフォーマット

YAML、Markdown、JSON コードをフォーマットするために、<Constant name="cloud" /> は [Prettier](https://prettier.io/) と連携します。Prettier は、保護されたプライマリ Git ブランチを含むすべてのブランチでフォーマットが可能です。<Constant name="cloud_ide" /> は保護されたブランチへのコミットをブロックするため、新しいブランチに変更をコミットするように促します。

1. `.yml`、`.md`、または `.json` ファイルを開きます。
2. コンソールセクション（**ファイルエディタ** の下）で [フォーマット] ボタンを選択し、**ファイルエディタ** でコードを自動フォーマットします。**コード品質** タブを使用してコードエラーを確認します。
3. 自動フォーマットが完了すると、結果を確認するメッセージが表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/prettier.gif" width="90%" title="Format YAML, Markdown, and JSON files using Prettier."/>

Prettier を使用して、YAML、Markdown、または JSON ファイルのフォーマットルールをカスタマイズするための設定ファイルを追加できます。IDE は優先順位に基づいて設定ファイルを検索します。たとえば、最初に `package.json` ファイル内の "prettier" キーを確認します。

優先順位とファイルの設定方法の詳細については、[Prettier のドキュメント](https://prettier.io/docs/en/configuration.html) を参照してください。`.prettierrc.json5`、`.prettierrc.js`、および `.prettierrc.toml` ファイルは現在サポートされていないことにご注意ください。

### Python をフォーマットする

Python コードをフォーマットするために、<Constant name="cloud" /> は [Black](https://black.readthedocs.io/en/latest/) と統合されています。Black は、妥協のない Python コードフォーマッタです。フォーマットは、保護されたプライマリ Git ブランチを含むすべてのブランチで利用できます。<Constant name="cloud_ide" /> は保護されたブランチへのコミットをブロックするため、新しいブランチに変更をコミットするように促します。

1. `.py` ファイルを開きます。
2. コンソールセクション（**ファイルエディタ** の下にあります）で [フォーマット] ボタンを選択し、**ファイルエディタ** でコードを自動フォーマットします。
3. 自動フォーマットが完了すると、結果を確認するメッセージが表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/python-black.gif" width="80%" title="Format Python files using Black."/>

## FAQs

<DetailsToggle alt_header="SQLFluff はいつ使用すればいいですか、また sqlfmt はいつ使用すればいいですか?">

SQLFluff と sqlfmt はどちらも SQL コードのフォーマットに使用されるツールですが、ユースケースによっては、いくつかの違いにより、どちらか一方が適している場合があります。<br />

SQLFluff は SQL コードのリンター兼フォーマッタです。コードを分析して潜在的な問題やバグを特定し、コーディング標準に準拠します。また、一連の [カスタマイズ可能](#customize-linting) なルールに従ってコードをフォーマットし、一貫したコーディングプラクティスを確保します。SQLFluff を使用すると、SQL コードのフォーマットを維持し、スタイルのベストプラクティスに従うこともできます。<br />

sqlfmt は SQL コード フォーマッタです。一連のフォーマット ルールに従って SQL コードを自動的にフォーマットしますが、これらのルールはカスタマイズできません。sqlfmt はコードの外観とレイアウトのみに焦点を当てており、インデント、改行、スペースの一貫性を確保します。sqlfmt はコードのエラーやバグを分析したり、コードのフォーマット以外のコーディングの問題を考慮したりはしません。 <br />

SQLFluff と sqlfmt は、好みや最適な方法に応じて使い分けることができます。

- SQLFluff を使用すると、コードの lint チェックとフォーマット（つまり、コードのエラーやバグを解析・修正し、スタイルをフォーマットすること）を実行できます。SQLFluff では、独自のルールを柔軟にカスタマイズできます。

- sqlfmt を使用すると、エラーやバグを解析せずに、コードのフォーマットのみを実行できます。sqlfmt はすぐに使用できるため、設定なしですぐに使用できます。

</DetailsToggle>

<DetailsToggle alt_header="`.sqlfluff` ファイルをネストできますか?">

最適なコード品質、一貫性、そしてスタイルを確保するため、プロジェクトのルートフォルダにメインの `.sqlfluff` 構成ファイルを 1 つ配置することを強くお勧めします。複数のファイルを配置すると、プロジェクト内でさまざまな SQL スタイルが使用される可能性があります。<br /><br />

ただし、dbt プロジェクトの特定のサブフォルダ内に、子の `.sqlfluff` 構成ファイルをカスタマイズして追加することもできます。<br /><br />`.sqlfluff` ファイルをサブフォルダにネストすると、SQLFluff はそのサブフォルダの構成ファイルで定義されたルールを、そのサブフォルダ内のすべてのファイルに適用します。親の `.sqlfluff` ファイルで指定されたルールは、サブフォルダ外にある他のすべてのファイルとフォルダに適用されます。この階層的なアプローチにより、プロジェクト全体の一貫性を維持しながら、カスタマイズされた lint ルールを適用できます。詳細については、[SQLFluff のドキュメント](https://docs.sqlfluff.com/en/stable/configuration.html#configuration-files) を参照してください。

</DetailsToggle>

<DetailsToggle alt_header="ターミナルから SQLFluff コマンドを実行できますか?">

現在、ターミナルからの SQLFluff コマンドの実行はサポートされていません。
</DetailsToggle>

<DetailsToggle alt_header="Studio IDE の外部で実行すると、SQLFluff の動作に一貫性がないのはなぜですか?">

- SQLFluff のバージョンが <Constant name="cloud_ide" /> のバージョンと一致していることを再度確認してください（lint 操作後の <b>コード品質</b> タブで確認できます）。<br /><br />
- 明確なルール違反があるにもかかわらず lint 操作が成功する場合は、一時的なモデルを使用して lint を実行していないことを確認してください。dbt v1.5 以前では、一時的なモデルは lint でサポートされていません。 

</DetailsToggle>

<DetailsToggle alt_header="dbt リンティングを使用する際の考慮事項は何ですか?">

現在、<Constant name="cloud_ide" /> は、一定のサイズと複雑さまでのファイルに対して lint または修正を実行できます。大きすぎるファイルに対して lint または修正を実行しようとすると、<Constant name="cloud" /> バックエンドの処理に 60 秒以上かかるため、「このファイルの lint を完了できません」というエラーが表示されます。

これを回避するには、モデルを小さなモデル（ファイル）に分割し、lint または修正が複雑にならないようにしてください。lint は修正よりも単純なため、ファイルが lint できても修正できない場合があります。

</DetailsToggle>

## 関連ドキュメント

- [ユーザーインターフェース](/docs/cloud/dbt-cloud-ide/ide-user-interface)
- [キーボードショートカット](/docs/cloud/dbt-cloud-ide/keyboard-shortcuts)
- [CIジョブにおけるSQLリンティング](/docs/deploy/continuous-integration#sql-linting)
