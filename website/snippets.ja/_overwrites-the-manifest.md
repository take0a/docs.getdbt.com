dbt は解析中に `manifest.json` ファイルを上書きします。つまり、`target/ directory` から `--state` を参照すると、保存されたマニフェストが見つからなかったことを示す警告が表示される場合があります。

<Lightbox src="/img/docs/reference/saved-manifest-not-found.png" title="Saved manifest not found error" /> 

次回のジョブ実行時に、dbt は問題を引き起こす一連の手順を実行します。まず、変更検出に使用する前に `target/manifest.json` を上書きします。その後、dbt が変更検出のために `target/manifest.json` を再度読み込もうとすると、以前の状態が既に上書き/消去されているため、変更は検出されません。

`--defer` や `state:modified` などの状態依存機能を使用する場合、`--state` と `--target-path` を同じパスに設定しないでください。そうしないと、べき等性が損なわれ、期待どおりに動作しなくなる可能性があります。