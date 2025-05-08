:::tip
dbt Core v1.10 以降では、`validate_macro_args` 動作変更フラグを使用して、マクロドキュメントで定義した引数を検証できます。有効にすると、dbt は以下の動作を行います。
- ドキュメント化された引数名がマクロ定義と一致しない場合に警告を表示します。
- `type` フィールドが [サポートされている形式](/reference/global-configs/behavior-changes#supported-types) に準拠していない場合に警告を表示します。

[マクロ引数の検証](/reference/global-configs/behavior-changes#macro-argument-validation) の詳細については、こちらをご覧ください。
:::
