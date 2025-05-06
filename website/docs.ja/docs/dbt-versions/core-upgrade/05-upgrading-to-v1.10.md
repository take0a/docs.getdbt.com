---
title: "v1.10へのアップグレード"
id: upgrading-to-v1.10
description: dbt Core v1.10 の新機能と変更点
displayed_sidebar: "docs"
---
 
## リソース

- dbt Core v1.10 の変更履歴（近日公開予定）
- [dbt Core CLI インストールガイド](/docs/core/installation-overview)
- [クラウドアップグレードガイド](/docs/dbt-versions/upgrade-dbt-version-in-cloud#release-tracks)

## アップグレード前に知っておくべきこと

dbt Labs は、すべてのバージョン 1.x に対して下位互換性を提供することに尽力しています。動作変更には、[動作変更フラグ](/reference/global-configs/behavior-changes#behavior-change-flags) が付与され、既存のプロジェクトへの移行期間が提供されます。アップグレード時にエラーが発生した場合は、[問題を報告](https://github.com/dbt-labs/dbt-core/issues/new) してお知らせください。

2024 年以降、dbt Cloud は、dbt Core の新しいバージョンの機能を [リリーストラック](/docs/dbt-versions/cloud-release-tracks) を通じて提供し、自動アップグレードを実施します。dbt Cloud で「最新」リリーストラックを選択した場合は、dbt Core v1.10 に含まれるすべての機能、修正、その他の機能を既にご利用いただけます。 「互換」リリーストラックを選択した場合、dbt Core v1.10 最終リリースの翌月次「互換」リリースからアクセスできるようになります。

dbt Core v1.8 以降をご利用の場合は、`dbt-core` と `dbt-<youradapter>` の両方を明示的にインストールすることをお勧めします。これは、dbt の将来のバージョンで必要になる可能性があります。例:

```sql
python3 -m pip install dbt-core dbt-snowflake
```

## 新機能と変更点

dbt Core v1.10 で利用可能な新機能

### `--sample` フラグ

大規模なデータセットは dbt のビルド時間を遅くし、開発者が新しいコードを効率的にテストすることを困難にする可能性があります。`run` コマンドと `build` コマンドで使用可能な [`--sample` フラグ](/docs/build/sample-flag) は、dbt をサンプルモードで実行することで、ビルド時間とウェアハウスコストを削減するのに役立ちます。このフラグは、時間ベースのサンプリングを使用してフィルタリングされた参照とソースを生成するため、開発者はモデル全体を構築することなく出力を検証できます。

### 従来の動作への変更の管理

dbt Core v1.10 では、[従来の動作への変更の管理](/reference/global-configs/behavior-changes) 用の新しいフラグが導入されました。`dbt_project.yml` の `flags` にそれぞれ `True` / `False` の値を設定することで、最近導入された変更を有効にするか（デフォルトでは無効）、成熟した変更を無効にするか（デフォルトでは有効）を選択できます。

これらの動作変更の詳細については、以下のリンクをご覧ください。

- (導入済み、デフォルトでは無効) [`validate_macro_args`](/reference/global-configs/behavior-changes#macro-argument-validation)フラグが `True` に設定されている場合、マクロ YAML に追加した引数 `type` 名がマクロ内の引数名と一致しない場合、または引数の型が [サポートされている型](/reference/global-configs/behavior-changes#supported-types) に従って有効でない場合、dbt は警告を発します。

## クイックヒット

- ソースの鮮度を表す [`loaded_at_query`](/reference/resource-properties/freshness#loaded_at_query) プロパティを指定して、ソースの `maxLoadedAt` タイムスタンプを生成するカスタム SQL を指定します（[組み込みクエリ](https://github.com/dbt-labs/dbt-adapters/blob/6c41bedf27063eda64375845db6ce5f7535ef6aa/dbt/include/global_project/macros/adapters/freshness.sql#L4-L16) では `loaded_at_field` が使用されます）。`loaded_at_field` 設定も指定されている場合は、`loaded_at_query` を定義することはできません。

- [`validate_macro_args`](/reference/global-configs/behavior-changes#macro-argument-validation) フラグを使用して、マクロ引数の検証を提供します。このフラグはデフォルトでは無効になっています。有効にすると、このフラグは、ドキュメント化されたマクロ引数名がマクロ定義内の引数名と一致するかどうかを確認し、サポートされている形式に照らして型を検証します。以前は、dbt は標準の引数型を強制せず、型フィールドをドキュメント専用として扱っていました。引数がドキュメント化されていない場合、dbt はマクロから引数を推測し、manifest.json ファイルに含めます。[サポートされている型](/reference/global-configs/behavior-changes#supported-types) の詳細については、こちらをご覧ください。
 