<Constant name="copilot" /> を使用すると、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) 内の SQL ファイル内で自然言語プロンプトを使用して SQL コードを直接生成できます。つまり、SQL ファイル全体を編集することなく、ファイルの特定の部分を書き換えたり追加したりできます。

このインテリジェントな AI ツールは、エラーを削減し、複雑なコードにも容易に対応し、貴重な時間を節約することで、SQL 開発を効率化します。<Constant name="copilot" /> の [プロンプトウィンドウ](#use-the-prompt-window) はキーボードショートカットでアクセスでき、反復的な SQL 生成や複雑な SQL 生成を簡単に処理するため、高度なタスクに集中できます。

Copilot のプロンプトウィンドウは、次のようなユースケースで使用できます。

- 高度な変換の作成
- 一括編集の効率的な実行
- 正規表現などの複雑なパターンの作成

### プロンプトウィンドウを使用する

キーボードショートカット Cmd+B (Mac) または Ctrl+B (Windows) を使用して <Constant name="copilot" /> の AI プロンプトウィンドウにアクセスし、以下の操作を行います。

#### 1. SQLを最初から生成する
- キーボードショートカットのCmd+B（Mac）またはCtrl+B（Windows）を使用して、SQLを最初から生成します。
- 指示を入力すると、自然言語を使用してニーズに合わせたSQLコードが生成されます。
- <Constant name="copilot" /> にコードの修正やSQLファイルの特定の部分の追加を依頼します。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/copilot-sql-generation-prompt.jpg" width="90%" title="dbt Copilot's prompt window accessible by keyboard shortcut Cmd+B (Mac) or Ctrl+B (Windows)" />

#### 2. 既存のSQLコードを編集する
- SQLコードの一部を選択し、Cmd+B (Mac) または Ctrl+B (Windows) を押すと、編集用のプロンプトウィンドウが開きます。
- これを使用して、ニーズに合わせて特定のコードスニペットを調整または変更します。
- <Constant name="copilot" /> にコードの修正やSQLファイルの特定部分の追加を依頼します。

#### 3. 変更を加える前に、差分ビューで変更内容を確認し、変更の影響を素早く評価します。
- 提案が生成されると、<Constant name="copilot" /> は視覚的な「差分」ビューを表示し、提案された変更と既存のコードを比較できるようにします。
  - **緑**: 提案を受け入れた場合に追加される新しいコードを示します。
  - **赤**: 提案された変更によって削除または置き換えられる既存のコードを強調表示します。

#### 4. 提案を承認または拒否する
- **承認**: 生成されたSQLが要件を満たしている場合は、「承認**」ボタンをクリックして、IDE内の`.sql`ファイルに変更を直接適用します。
- **拒否**: 提案が要求/プロンプトと一致しない場合は、「拒否**」をクリックして、生成されたSQLを変更せずに破棄し、やり直します。

#### 5. コードの再生成
- 再生成するには、キーボードの **Esc** キーを押すか、ポップアップの「拒否」ボタンをクリックします。これにより、生成されたコードが削除され、カーソルがプロンプトのテキストエリアに戻ります。
- プロンプトを更新し、**Enter** キーを押して再度生成を試みます。もう一度 **Esc** キーを押すと、ポップオーバーが完全に閉じます。

提案を受け入れたら、プロンプトウィンドウを使用して追加の SQL コードを生成し、変更をブランチにコミットできます。

<Lightbox src="/img/docs/dbt-cloud/cloud-ide/copilot-sql-generation.gif" width="100%" title="Edit existing SQL code using dbt Copilot's prompt window accessible by keyboard shortcut Cmd+B (Mac) or Ctrl+B (Windows)" />
