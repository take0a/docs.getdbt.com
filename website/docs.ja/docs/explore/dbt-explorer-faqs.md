---
title: "dbt Catalog FAQs"
sidebar_label: "dbt Catalog FAQs"
description: "Learn more with the FAQs about dbt Catalog, how it works, how to interact with it, and more."
---

[<Constant name="explorer" />](/docs/explore/explore-projects) は、<Constant name="cloud" /> の新しいナレッジベースとリネージ可視化エクスペリエンスです。企業のデータ資産全体をインタラクティブかつ高レベルで可視化し、リネージの理解と改善に必要なコンテキストを深く掘り下げることで、チームが意思決定に使用するデータに信頼を置けるようになります。

## 概要

<Expandable alt_header="dbt Catalog はデータ品質にどのように役立ちますか?" >

<Constant name="explorer" /> を使用すると、データソースからレポートレイヤーに至るまで、系統全体を簡単かつ直感的に把握できるため、パイプラインのトラブルシューティング、改善、最適化が可能になります。プロジェクトの推奨事項やモデルパフォーマンス分析などの組み込み機能により、資産全体にわたって適切なテストとドキュメントが確実に網羅され、実行速度の遅いモデルを迅速に特定して修正できます。列レベルの系統を使用すると、テーブルの変更による下流への潜在的な影響を迅速に特定したり、遡ってインシデントの根本原因を迅速に把握したりできます。<Constant name="explorer" /> は、データ品質をプロアクティブに改善するために必要な分析情報をチームに提供し、パイプラインのパフォーマンスを維持し、データの信頼性を強固に保ちます。

</Expandable>

<Expandable alt_header="dbt Catalog の価格設定はどうなっていますか?" >

<Constant name="explorer" /> は、すべての <Constant name="cloud" /> [Enterprise 層および Starter プラン](https://www.getdbt.com/) のすべてのリージョンおよびデプロイメントタイプで一般提供されています。<Constant name="explorer" /> の一部の機能（プロジェクト推奨、マルチプロジェクト系統、列レベルの系統など）は、Enterprise プランおよび Enterprise+ プランでのみご利用いただけます。

<Constant name="explorer" /> には、開発者ライセンスおよび読み取り専用ライセンスを持つユーザーがアクセスできます。

</Expandable>

<Expandable alt_header="dbt Docs に何が起こったのでしょうか?" >

<Constant name="explorer" /> は、<Constant name="cloud" /> のお客様のデフォルトのドキュメント エクスペリエンスです。dbt Docs は引き続き利用可能ですが、<Constant name="explorer" /> と同じ速度、メタデータ、可視性は提供されず、レガシー機能になります。

</Expandable>

## dbtカタログの仕組み

<Expandable alt_header="dbt Catalog をオンプレミスで使用したり、セルフホスト型の dbt Core デプロイメントで使用したりできますか?" >

いいえ。<Constant name="explorer" /> とそのすべての機能は、<Constant name="cloud" /> ユーザー エクスペリエンスとしてのみ利用できます。<Constant name="explorer" /> は、<Constant name="cloud" /> プロジェクトとその実行からのメタデータを反映します。

</Expandable>

<Expandable alt_header="dbt Catalog は dbt 環境をどのようにサポートしますか?" >

<Constant name="explorer" /> は、探索するプロジェクトごとに、本番環境またはステージング環境の [デプロイメント環境](/docs/deploy/deploy-environments) をサポートします。デフォルトでは、プロジェクトの最新の本番環境またはステージング環境の状態になります。ユーザーは、<Constant name="cloud" /> プロジェクトごとに、本番環境とステージング環境をそれぞれ 1 つずつしか割り当てることができません。

開発環境 (<Constant name="cloud_cli" /> および <Constant name="cloud_ide" />) のサポートは近日中に開始される予定です。

</Expandable>

<Expandable alt_header="カタログを使い始めるにはどうすればいいですか? どのように更新されますか?" >

<Constant name="cloud" /> 上部のナビゲーションバーから [**Explore**] を選択するだけです。<Constant name="explorer" /> は、指定されたプロジェクトの環境（デフォルトでは本番環境）で <Constant name="cloud" /> が実行されるたびに自動的に更新されます。環境内で実行する dbt コマンドは、<Constant name="explorer" /> 内のメタデータを生成および更新するため、環境のジョブ内で正しいコマンドの組み合わせを実行するようにしてください。詳細については、[メタデータの生成](/docs/explore/explore-projects#generate-metadata) を参照してください。

</Expandable>

<Expandable alt_header="dbt 系統を外部システムまたはカタログにエクスポートすることは可能ですか?" >

はい。<Constant name="explorer" /> を動かす系統は、Discovery API を通じても利用できます。

</Expandable>

<Expandable alt_header="dbt Catalog はどのようにしてサードパーティのツールと統合してエンドツーエンドの系統を表示するのでしょうか?" >

<Constant name="explorer" /> は、dbt プロジェクト内で定義されたすべての系統を反映しています。<Constant name="explorer" /> のビジョンは、<Constant name="cloud" /> に統合されたデータローダー（ソース）や BI/アナリティクスツール（エクスポージャー）などの外部ツールからの追加メタデータを、すべて <Constant name="cloud" /> プロジェクトの系統にシームレスに組み込むことです。

</Expandable>

<Expandable alt_header="dbt Catalogで以前は表示されていたデータが消えたのはなぜですか?" >

<Constant name="explorer" /> は、メタデータを更新するジョブが実行されていない場合、3か月後に古いメタデータを自動的に削除します。これを回避するには、必要なコマンドを使用して、3か月よりも頻繁にジョブを実行するようにスケジュールを設定してください。

</Expandable>

## 主な特徴

<Expandable alt_header="dbt Catalog はマルチプロジェクトの検出 (dbt Mesh) をサポートしていますか?" >

はい。詳細については、[複数のプロジェクトの探索](/docs/explore/explore-multiple-projects)を参照してください。

</Expandable>

<Expandable alt_header="dbt Catalog はどのような検索機能をサポートしていますか?" >

リソース検索機能には、キーワード、部分文字列（あいまい検索）、および「OR」などの集合演算子の使用が含まれます。また、系統検索ではdbtセレクタの使用がサポートされています。詳細については、[キーワード検索](/docs/explore/explore-projects#search-resources)をご覧ください。

</Expandable>

<Expandable alt_header="現在実行中のジョブのモデル実行情報を表示できますか?" >

<Constant name="cloud" /> は、ジョブの実行後にパフォーマンス チャートとメトリックを更新します。

</Expandable>

<Expandable alt_header="1 か月以内に成功したモデル実行の数を分析できますか?" >

月別に構築されたモデルのグラフは、<Constant name="cloud" /> ダッシュボードで確認できます。

</Expandable>

<Expandable alt_header="モデルまたは列の説明を dbt 内で編集できますか?" >

はい。現在、dbt プロジェクト内の YAML ファイルを変更することで、<Constant name="cloud_ide" /> または <Constant name="cloud_cli" /> の説明を編集できます。将来的には、<Constant name="explorer" /> でも説明を編集できる方法がさらに増える予定です。

</Expandable>

<Expandable alt_header="推奨事項はどこから来ますか？カスタマイズできますか？" >

推奨事項は、`dbt_project_evaluator` パッケージのベストプラクティスルールをほぼ反映しています。現時点では、推奨事項をカスタマイズすることはできません。将来的には、<Constant name="explorer" /> で推奨事項のカスタマイズ機能（プ​​ロジェクトコード内など）がサポートされる予定です。

</Expandable>

## 列レベルの系統

<Expandable alt_header="dbt Catalog の列レベルの系統の最適な使用例は何ですか?" >

<Constant name="explorer" /> の列レベルの系統は、次のような多くのデータ開発ワークフローの改善に使用できます。

- **監査** - dbt プロジェクト内でデータがどのように移動し、使用されているかを視覚化します。
- **根本原因** - データ品質の問題を検出して解決するまでの時間を短縮し、ソースを遡って追跡します。
- **影響分析** - 変換と使用状況を追跡し、利用者に問題が生じないようにします。
- **効率** - 不要な列を削除して、コストとデータチームのオーバーヘッドを削減します。

</Expandable>

<Expandable alt_header="モデル間で列名が異なる場合でも、列レベルの系統は機能し続けますか?" >

はい。列レベルの系統は、dbt プロジェクト内の列のインスタンス間での名前の変更を処理できます。

</Expandable>

<Expandable alt_header="複数のプロジェクトで同じ列定義を活用できますか?" >

いいえ。プロジェクト間の列系統は、パブリック モデルがプロジェクト間でどのように使用されているかを表示するという意味ではサポートされていますが、列レベルではサポートされていません。

</Expandable>


<Expandable alt_header="列の説明は下流系統に自動的に伝播できますか?" >

はい、passthrough または rename としてラベル付けされた再利用列は、ソース列と上流のモデル列から説明を継承します。つまり、ソース列と上流のモデル列は、変換されない場合、その説明を下流に伝播するため、手動で説明を定義する必要はありません。詳しくは、[継承された列の説明](/docs/explore/column-level-lineage#inherited-column-descriptions)をご覧ください。

</Expandable>

<Expandable alt_header="列レベルの系統は開発タブでも利用できますか?" >

現時点ではそうではありませんが、将来的には <Constant name="cloud" /> の機能全体に列レベルの認識を組み込む予定です。

</Expandable>

## 可用性、アクセス、および権限

<Expandable alt_header="開発者以外のユーザーが dbt Catalog を操作するにはどうすればよいですか?" >

読み取り専用ユーザーは、<Constant name="explorer" /> でメタデータを利用できます。今後、アナリストや技術に詳しくない貢献者向けに、よりカスタマイズされたエクスペリエンスと探索手段が提供される予定です。

</Expandable>

<Expandable alt_header="dbt Catalog には特定の dbt プランが必要ですか?" >

<Constant name="explorer" /> は、dbt Starter プランおよびすべての Enterprise プランでご利用いただけます。<Constant name="explorer" /> の一部の機能（プロジェクトの推奨事項、複数プロジェクトの系統、列レベルの系統など）は、Enterprise プランおよび Enterprise+ プランでのみご利用いただけます。

</Expandable>

<Expandable alt_header="dbt Core ユーザーはこれらの新しい dbt Catalog 機能を活用できるようになりますか?" >

いいえ。<Constant name="explorer" /> は <Constant name="cloud" /> 専用の製品エクスペリエンスです。

</Expandable>

<Expandable alt_header="読み取り専用ライセンスを使用して dbt Catalog にアクセスすることは可能ですか?" >

はい、読み取り専用アクセス権を持つユーザーは <Constant name="explorer" /> を使用できます。<Constant name="explorer" /> で利用できる具体的な機能は、<Constant name="cloud" /> プランによって異なります。

</Expandable>

<Expandable alt_header="役に立つ dbt Catalog のコンテンツを dbt 外部の人と共有する簡単な方法はありますか?" >

ビューを埋め込んで共有する機能は、将来の潜在的な機能として評価されています。

</Expandable>

<Expandable alt_header="dbt Catalog は dbt 内の他の領域からアクセスできますか?" >

はい、[さまざまな <Constant name="cloud" /> 機能から <Constant name="explorer" /> にアクセス](/docs/explore/access-from-dbt-cloud) できるため、プロジェクト内のリソースと系統間をシームレスに移動できます。

<Constant name="explorer" /> にアクセスする主な方法は、ナビゲーションの [**Explore**] リンクを使用することですが、[<Constant name="cloud_ide" />](/docs/explore/access-from-dbt-cloud#dbt-cloud-ide)、[ジョブの系統タブ](/docs/explore/access-from-dbt-cloud#lineage-tab-in-jobs)、[ジョブのモデルタイミングタブ](/docs/explore/access-from-dbt-cloud#model-timing-tab-in-jobs) からもアクセスできます。

</Expandable>
