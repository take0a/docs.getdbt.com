dbt Copilot を使用すると、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) 内のボタンをクリックするだけで、ドキュメント、テスト、メトリクス、セマンティックモデル [リソース](/docs/build/projects) を生成できるため、時間を節約できます。この AI 機能にアクセスして使用するには、次の手順を実行します。

1. <Constant name="cloud_ide" /> に移動し、**ファイルエクスプローラー** で SQL モデルファイルを選択します。
2. **コンソール** セクション (**ファイルエディタ** の下) で、**dbt Copilot** をクリックして、利用可能な AI オプションを表示します。
3. YAML 構成を生成するために利用可能なオプション (**ドキュメントの生成**、**テストの生成**、**セマンティックモデルの生成**、または **メトリクスの生成**) を選択します。同じモデルに対して複数の YAML 構成を生成するには、各オプションを個別にクリックします。 dbt Copilot は、YAML 構成を同じファイルにインテリジェントに保存します。
   - メトリクスを生成するには、まずセマンティックモデルを定義する必要があります。
   - 定義したら、**dbt Copilot** をクリックし、**Generate Metrics** を選択します。
   - 生成するメトリクスを説明するプロンプトを入力し、Enter キーを押します。
   - 生成されたコードを **Accept** または **Reject** します。
4. AI によって生成されたコードを確認します。必要に応じてコードを更新または修正できます。
5. **Save As** をクリックします。**Version control** セクションにファイルの変更が表示されます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/dbt-copilot-doc.gif" width="100%" title="Example of using dbt Copilot to generate documentation in the IDE" />
