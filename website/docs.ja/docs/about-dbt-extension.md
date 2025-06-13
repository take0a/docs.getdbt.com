---
title: About the dbt VS Code extension
id: about-dbt-extension
description: "Bring all the speed and power of the dbt Fusion engine to your local development workflow."
sidebar_label: "About the dbt VS Code extension"
pagination_next: "docs/install-dbt-extension"
---

# dbt VS Code拡張機能について <Lifecycle status="beta" />

dbt拡張機能は、VS Codeに超高速、インテリジェント、そしてコスト効率に優れたdbt開発エクスペリエンスをもたらします。

これは、ローカル開発中に新しいdbt Fusionエンジンのパワーをすべて活用できる唯一の方法です。

_時間とリソースを節約_、ほぼ瞬時の解析、ライブエラー検出、強力なIntelliSense機能などを活用します。

_ローカルdbt開発向けにゼロから設計されたシームレスでエンドツーエンドのdbt開発エクスペリエンスで、_開発フローを維持_します。

_これはパブリックベータリリースです。より広範な一般公開（GA）リリースの前に動作が変更される場合があります。_

## 生産性向上機能

以下の拡張機能は、より多くの作業をより速く、より効率的に行うのに役立ちます。

- **[ライブエラー検出](#live-error-detection):** ウェアハウスにアクセスすることなく、SQLコードを自動的に検証し、エラーを検出して警告を表示します。これには、dbtエラー（無効な `ref` など）とSQLエラー（無効な列名やSQL構文など）の両方が含まれます。
- **[超高速解析時間](#lightning-fast-parse-times):** 大規模なプロジェクトでも、dbt Coreよりも最大30倍高速に解析できます。
- **[強力なIntelliSense](#powerful-intellisense):** SQL関数、モデル名、列、マクロなどを自動補完します。
- **[インスタントリファクタリング](#instant-refactoring):** モデルまたは列の名前を変更すると、プロジェクト全体で参照が更新されます。
- **[定義へ移動](#go-to-definition-and-reference):** ワンクリックで任意の `ref`、マクロ、モデル、または列の定義に移動できます。特に、多数のモデルとマクロを含む大規模プロジェクトで便利です。
- **[ホバーインサイト](#hover-insights):** コードを離れることなく、テーブル、列、関数のコンテキストを確認できます。SQL 要素にマウスを合わせるだけで、列名やデータ型などの詳細が表示されます。
- **[ライブ CTE プレビュー](#live-preview-for-models-and-ctes):** dbt モデル内から直接 CTE の出力をプレビューできるため、検証とデバッグを高速化できます。
- **[コンテキスト内の豊富なリネージ](#rich-lineage-in-context):** 開発中に、コンテキストを切り替えたりフローを中断したりすることなく、列またはテーブルレベルでリネージを確認できます。
- **[コンパイル済みコードを表示](#view-compiled-code):** モデルがビルドする SQL コードを dbt コードと一緒にライブビューで表示します。
- **[柔軟なビルド](#build-flexibly):** コマンドパレットを使用して、複雑なセレクターを含むモデルを構築します。
 
### Live error detection

SQLコードを自動的に検証し、ウェアハウスにアクセスすることなくエラーを検出し、警告を表面化させます。

- 以下のエラーについて診断情報（赤い波線）を表示します。
  - 構文エラー（カンマの欠落、キーワードのスペルミスなど）。
  - 列名が無効または欠落している（例：`select not_a_column from {{ ref('real_model') }}`）。
  - `group by`句が欠落している、またはグループ化も集計もされていない列。
  - 関数名または引数が無効。
- 赤い波線にマウスポインターを合わせるとエラーが表示されます。
- 診断情報の詳細は「問題」をご覧ください。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/live-error-detection.webm" type="video/webm" />
</video>

### Lightning-fast parse times

最大規模のプロジェクトでも、dbt Core より最大 30 倍高速に解析します。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/zoomzoom.webm" type="video/webm" />
</video>

### Powerful IntelliSense

SQL 関数、モデル名、列、マクロなどを自動補完します。

使用方法:
- `ref` および `source` 呼び出しを自動補完します。たとえば、`{{ ref(` または `{{ source(` と入力すると、利用可能なリソースとその型の一覧が表示され、関数呼び出しが補完されます。
- 方言固有の関数名を自動補完します。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/intellisense.webm" type="video/webm" />
</video>

### Instant refactoring

モデル名の変更:
- ファイルツリー内のファイルを右クリックし、**名前の変更** を選択します。
- ファイル名を変更すると、リファクタリングの変更を行うかどうかを確認するメッセージが表示されます。
- 変更を適用するには**OK** を選択するか、リファクタリングのプレビューを表示するには**プレビューの表示** を選択します。
- 変更を適用すると、`ref` が更新され、更新されたモデル名が使用されるようになります。

列名の変更:
- 列のエイリアスを右クリックし、**シンボル名の変更** を選択します。
- 列名を変更すると、リファクタリングの変更を行うかどうかを確認するメッセージが表示されます。
- 変更を適用するには**OK** を選択するか、リファクタリングのプレビューを表示するには**プレビューの表示** を選択します。
- 変更を適用すると、列への下流の参照が更新され、新しい列名が使用されるようになります。

注: スナップショット、または .yml ファイルで定義されたリソースでは、モデルと列の名前変更はまだサポートされていません。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/refactor.webm" type="video/webm" />
</video>

### Go-to-definition and reference

`ref`、マクロ、モデル、または列の定義にワンクリックで移動できます。特に、多数のモデルやマクロを含む大規模プロジェクトで便利です。

使用方法:
- Command キーまたは Ctrl キーを押しながらクリックすると、識別子の定義に移動します。
- 識別子を右クリックして、**定義へ移動** または **参照へ移動** を選択することもできます。
- CTE 名、列名、`*`、マクロ名、および dbt `ref()` および `source()` 呼び出しをサポートします。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/go-to-definition.webm" type="video/webm" />
</video>

### Hover insights

コードから離れることなく、テーブル、列、関数のコンテキストを確認できます。SQL 要素にマウスオーバーするだけで、列名やデータ型などの詳細が表示されます。

使用方法:
- `*` にマウスオーバーすると、列とその型の拡張リストが表示されます。
- 列名またはエイリアスにマウスオーバーすると、その型が表示されます。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/hover-insights.webm" type="video/webm" />
</video>

### Live preview for models and CTEs

CTE の出力またはモデル全体をエディター内から直接プレビューできるため、検証とデバッグを迅速に行うことができます。

使用方法:
- **テーブルアイコン** をクリックするか、キーボードショートカット `cmd+enter` (macOS) / `ctrl+enter` (Windows/Linux) を使用してクエリ結果をプレビューします。
- CTE の結果をプレビューするには、**Preview CTE** コードレンズをクリックします。
- 結果は下部パネルの **Query Results** タブに表示されます。
- プレビューテーブルは並べ替え可能で、結果はタブを閉じるまで保存されます。
- SQL の範囲を選択して、特定の SQL スニペットの結果をプレビューすることもできます。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/preview-cte.webm" type="video/webm" />
</video>

### Rich lineage in context

開発中に列レベルまたはテーブルレベルで系統図を確認できます。コンテキストの切り替えやフローの中断は発生しません。

テーブル系統図の表示:
- エディターで **Lineage** タブを開きます。現在開いているファイルにフォーカスが当てられているテーブル系統図が反映されます。
- ノードをダブルクリックすると、エディターでファイルが開きます。
- dbt プロジェクト内のファイル間を移動すると、系統図ペインが更新されます。
- ノードを右クリックすると、DAG が更新されるか、ノードの列系統図が表示されます。

列系統図の表示:
- ファイル名またはモデルファイルの SQL コンテンツを右クリックします。
- **dbt: View Lineage** --> **Show column lineage** を選択します。
- 系統図を表示する列を選択します。
- ノードをダブルクリックすると、DAG セレクターが更新されます。
- 系統図ウィンドウで、`column:` プレフィックスと列名を追加することで、列セレクターを使用することもできます。
  - たとえば、`stg_payments` モデルの `AMOUNT` 列の系統が必要な場合は、`+model.jaffle_shop.stg_payments+` を `+column:model.jaffle_shop.stg_payments.AMOUNT+` に編集します。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/lineage.webm" type="video/webm" />
</video>

### View compiled code

モデルが構築するSQLコードを、dbtコードと並べてリアルタイムで確認できます。

使用方法：
- **コードアイコン**をクリックすると、コンパイル済みコードとソースコードを並べて表示できます。
- ソースコードを保存すると、コンパイル済みコードが更新されます。
- dbtマクロをクリックすると、対応するコンパイル済みコードにフォーカスが移動します。
- コンパイル済みコードブロックをクリックすると、対応するソースコードにフォーカスが移動します。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/compiled-code.webm" type="video/webm" />
</video>

### Build flexibly

コマンドパレットを使用すると、複雑なセレクターを使ったモデルを素早く構築できます。

使用方法:
- **dbt アイコン** をクリックするか、キーボードショートカット `cmd+shift+enter` (macOS) / `ctrl+shift+enter` (Windows/Linux) を使用してクイックピックメニューを起動します。
- 実行するコマンドを選択します。

<video width="100%" height="100%" playsinline muted controls>
  <source src="/img/docs/extension/build-flexibly.webm" type="video/webm" />
</video>

## 拡張機能の使用

この拡張機能を使用するには、dbt 環境で dbt Fusion エンジンを使用している必要があります。ご利用資格とアップグレードの詳細については、[Fusion のドキュメント](/docs/fusion/about-fusion) をご覧ください。

インストールが完了すると、dbt プロジェクト ディレクトリ内の `.sql` ファイルまたは `.yml` ファイルを開くと、dbt 拡張機能が自動的にアクティブになります。

## 構成

インストール後、開発ワークフローに合わせて拡張機能を設定することをお勧めします。

1. `Ctrl+,` (Windows/Linux) または `Cmd+,` (Mac) を押して、VS Code の設定画面を開きます。
2. `dbt` を検索します。このページで、ニーズに合わせて拡張機能の設定オプションを調整できます。

## FAQs

**モノレポでdbt拡張機能を使用できますか？**

dbt拡張機能は、ワークスペースのルートフォルダに`dbt_project.yml`ファイルが見つからないとアクティブになりません。モノレポで開発する場合は、[.code-workspace](https://code.visualstudio.com/docs/editing/workspaces/workspaces#_singlefolder-workspaces)ファイルを使用して、dbtプロジェクトフォルダ用のワークスペースを作成することを検討してください。これは、エディタで`Add Folder to Workspace`コマンドを実行するだけで簡単に実行できます。


## 既知の制限事項

dbt 拡張機能の既知の制限事項は以下のとおりです。

- **リモート開発:** dbt 拡張機能は、SSH 経由のリモート開発セッションをまだサポートしていません。今後のリリースでサポートされる予定です。リモート開発の詳細については、[リモート開発と GitHub Codespaces のサポート](https://code.visualstudio.com/api/advanced-topics/remote-extensions) および [Visual Studio Code サーバー](https://code.visualstudio.com/docs/remote/vscode-server) を参照してください。

- **YAML ファイルの操作:** 現在、dbt 拡張機能には YAML ファイルの操作に関して以下の制限があります。
  - YAML ファイルで定義されたノード (スナップショットなど) では、定義への移動はサポートされていません。
  - モデルや列の名前を変更しても、YAML ファイル内の参照は更新されません。
  - dbt拡張機能の今後のリリースでは、これらの制限事項に対処します。

- **モデルの名前変更:** モデルファイルの名前が変更されると、dbt拡張機能は変更を適用し、名前変更されたモデルを参照するすべての`ref()`呼び出しを更新します。VS Codeの言語サーバークライアントの制限により、これらの編集ファイルを自動保存することはできません。そのため、モデルファイルの名前を変更すると、プロジェクトでコンパイラエラーが発生する可能性があります。これらのエラーを修正するには、dbt拡張機能によって編集された各ファイルを手動で保存するか、**ファイル** --> **すべて保存** をクリックして編集したすべてのファイルを保存する必要があります。


## サポート

dbtプラットフォームをご利用のお客様は、dbt Labsサポート（[support@getdbt.com](mailto:support@getdbt.com)）までお問い合わせください。また、担当のアカウントマネージャーに直接ご連絡いただくことも可能です。

dbtプラットフォームをご利用でない組織の方は、[dbtコミュニティSlack](https://www.getdbt.com/community/join-the-community)をご利用ください。ご質問やご意見は、ぜひお気軽にお問い合わせください。

拡張機能の継続的な改善に努めておりますので、皆様からのフィードバックをお待ちしております。
