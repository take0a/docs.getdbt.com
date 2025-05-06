---
title: "dbt Cloudでドキュメントを作成して表示する"
id: "build-and-view-your-docs"
description: "ジョブを実行すると、プロジェクト ドキュメントが自動的に生成されます。"
pagination_next: null
---

dbt Cloud を使用すると、プロジェクトとデータ プラットフォームのドキュメントを生成できます。ドキュメントはジョブの実行が完全に成功すると新しい情報で自動的に更新されるため、正確性と関連性が確保されます。

dbt Cloud のデフォルトのドキュメント エクスペリエンスは [dbt Explorer](/docs/collaborate/explore-projects) で、[Team プランまたは Enterprise プラン](https://www.getdbt.com/pricing/) でご利用いただけます。[dbt Explorer](/docs/collaborate/explore-projects) を使用して、プロジェクトのリソース（モデル、テスト、メトリックなど）とそのリネージを表示し、最新の本番環境の状態をより深く理解できます。

設定の詳細については、[ドキュメント](/docs/build/documentation) を参照してください。

この変更により、[dbt Docs](#dbt-docs) は dbt Cloud のレガシー ドキュメント機能となります。dbt Docs は引き続きアクセス可能で基本的なドキュメントを提供しますが、dbt Explorer と同じ速度、メタデータ、可視性は提供されません。 dbt Docs は、dbt Cloud 開発者プランまたは dbt Core ユーザーが利用できます。

## ドキュメントジョブの設定

dbt Explorer は、本番環境またはステージング環境で各ジョブを実行するたびに生成される [メタデータ](/docs/collaborate/explore-projects#generate-metadata) を使用することで、常に最新のプロジェクト結果を保持します。より詳細なメタデータを表示するには、ジョブ設定の編集時または新規ジョブの作成時に、dbt Cloud でジョブのドキュメントを設定できます。

ジョブの実行時に [メタデータを生成](/docs/collaborate/explore-projects#generate-metadata) するようにジョブを構成します。dbt Explorer でモデル、ソース、スナップショットの列と統計情報を表示する場合は、この手順が必要です。

ドキュメントを生成するジョブを設定するには:

1. 左上の **Deploy**  をクリックし、**Jobs** を選択します。
2. 新しいジョブを作成するか、既存のジョブを選択して **Settings** をクリックします。
3. **Execution Settings** で **Generate docs on run** を選択し、**Save** をクリックします。

   <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/documentation-job-execution-settings.png" width="100%" title="Setting up a job to generate documentation"/>

*注: dbt Docs をご利用の場合は、ジョブの実行時にドキュメントを生成するように設定し、そのジョブをプロジェクトに手動でリンクする必要があります。[プロジェクト ドキュメントの設定](#configure-project-documentation)に進み、このジョブの実行時にプロジェクトがドキュメントを生成するようにしてください。*

[`dbt docs generate` コマンド](/reference/commands/cmd-docs)をジョブ実行ステップのコマンドリストに追加することもできます。ただし、実行ステップにコマンドを追加した場合と、[**実行時にドキュメントを生成**] チェックボックスをオンにしてジョブを構成した場合とでは、結果が異なる場合があります。

以下のオプションと結果を確認してください。

| Options | Outcomes |
|--------| ------- |
| **Select checkbox** | **Generate docs on run** チェックボックスをオンにすると、ジョブが実行されるたびに更新されたプロジェクトドキュメントが自動的に生成されます。ジョブ内の特定のステップが失敗した場合でも、後続のステップがすべて成功すればジョブ自体は成功します。 |
| **Add as a run step** | ジョブ実行ステップのコマンドリストに「dbt docs generate」を任意の順序で追加します。ジョブ内の特定のステップが失敗した場合、ジョブは失敗し、後続のすべてのステップはスキップされます。 |

:::tip Tip &mdash; ドキュメント作成のみのジョブ

本番環境ジョブの最後にドキュメント作成のみのジョブを作成してスケジュールするには、**コマンド** セクションに `dbt compile` コマンドを追加します。

:::

## dbt Docs

dbt Docsは、開発者プランまたはdbt Coreユーザーでご利用いただけます。`dbt docs generate`コマンドを使用して、dbtプロジェクトからウェブサイトを生成します。モデル、テスト、リネージといったプロジェクトのリソースを一元的に確認できる場所を提供し、ウェアハウス内のデータを理解するのに役立ちます。

### プロジェクトドキュメントの設定

前のセクションで設定したジョブの実行時にドキュメントを生成するように、プロジェクトドキュメントを設定します。プロジェクト設定で、そのプロジェクトのドキュメントアーティファクトを生成するジョブを指定します。この設定を行うと、次回以降のジョブ実行時にドキュメント生成のステップが自動的に含まれるようになります。

1. dbt Cloud の左側のメニューでアカウント名をクリックし、 **Account settings** を選択します。
2. **Projects** に移動し、ドキュメントを生成するプロジェクトを選択します。
3. **Edit** をクリックします。
4. **Artifacts** で、実行時にドキュメントを生成するジョブを選択し、**Save** をクリックします。

   <Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/documentation-project-details.png" width="100%" title="Configuring project documentation"/>

:::tip より充実したドキュメント作成エクスペリエンスを実現するには、dbt Explorer をご利用ください。
より充実したインタラクティブなエクスペリエンスを実現するには、[dbt Explorer](/docs/collaborate/explore-projects) をお試しください。[Team プランまたは Enterprise プラン](https://www.getdbt.com/pricing/) でご利用いただけます。DAG のマップレイヤー、キーワード検索、IDE との連携、モデルのパフォーマンス、プロジェクトの推奨事項など、さまざまな機能を備えています。
:::

### ドキュメントの生成

dbt Cloud IDE でドキュメントを生成するには、dbt Cloud IDE の **Command Bar** で `dbt docs generate` コマンドを実行します。このコマンドは、IDE セッションで開発中の dbt プロジェクトのドキュメントを生成します。

dbt Cloud IDE で `dbt docs generate` コマンドを実行した後、ファイルツリーの上にあるアイコンをクリックすると、新しいブラウザウィンドウに最新バージョンのドキュメントが表示されます。

### ドキュメントの表示

プロジェクトのドキュメントを生成するジョブを設定したら、ナビゲーションで **Explore** をクリックし、**dbt Docs** をクリックします。プロジェクトのドキュメントが開きます。このリンクから、dbt Cloud でプロジェクトのドキュメントの最新バージョンをいつでも見つけることができます。

生成されたドキュメントには、常に最後に完全に成功した実行が表示されます。つまり、テストなどのタスクが失敗した場合、今回の実行ではドキュメントの変更は表示されません。完全に成功した実行がない場合は、ドキュメントの変更は表示されません。

dbt Cloud IDE を使用すると、コードの開発中でも dbt プロジェクトの [ドキュメント](/docs/build/documentation) を表示できます。このワークフローにより、変更が本番環境にリリースされる前に、プロジェクトで生成されたドキュメントがどのように表示されるかを確認できます。

## 関連ドキュメント
- [ドキュメント](/docs/build/documentation)
- [dbt Explorer](/docs/collaborate/explore-projects)