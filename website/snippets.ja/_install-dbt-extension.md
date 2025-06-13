
VS Code で:

1. エディターの **Extensions** タブに移動し、「dbt」を検索します。発行元が `dbtLabsInc` または `dbt Labs Inc` の拡張機能を見つけます。[**Install**] をクリックします。
    <Lightbox src="/img/docs/extension/extension-marketplace.png" width="60%" title="Search for the extension"/>
2. VS Code環境でdbtプロジェクトを開きます（まだ開いていない場合は）。現在のワークスペースに追加されていることを確認してください。エディターのステータスバーに**dbt Extension**ラベルが表示されていれば、拡張機能は正常にインストールされています。この**dbt Extension**ラベルにマウスポインターを合わせると、拡張機能に関する診断情報が表示されます。
    <Lightbox src="/img/docs/extension/dbt-extension-statusbar.png" width="60%" title="If you see the 'dbt Extension` label, the extension is activated"/>
3. dbt 拡張機能がアクティブ化されると、オペレーティング システムに適した dbt 言語サーバーのダウンロードが自動的に開始されます。
    <Lightbox src="/img/docs/extension/extension-lsp-download.png" width="60%" title="The dbt Language Server will be installed automatically"/>
4. dbt Fusion エンジンがまだマシンにインストールされていない場合は、拡張機能によってダウンロードとインストールを求めるメッセージが表示されます。通知に表示される手順に従ってインストールを完了してください。
    <Lightbox src="/img/docs/extension/install-dbt-fusion-engine.png" width="60%" title="Follow the prompt to install the dbt Fusion engine"/>
5. これで設定は完了です。dbt拡張機能の使い方について詳しくは、[dbt拡張機能について](/docs/about-dbt-extension)をご覧ください。
    <Lightbox src="/img/docs/extension/kitchen-sink.png" width="60%" title="Showing lineage and compiled code in the extension"/>
