---
title: "キャッシュ"
id: "cache"
sidebar: "キャッシュ"
---

### キャッシュへのデータ投入

実行開始時に、dbt はリソース（モデルなど）をマテリアライズする可能性のあるすべてのスキーマ内のすべてのオブジェクトに関するメタデータをキャッシュします。デフォルトでは、dbt はプロジェクトに関連するすべてのスキーマの情報をリレーショナルキャッシュにデータ投入します。

この動作をオプションで変更する方法は 2 つあります。
- `POPULATE_CACHE` (デフォルト: `True`): キャッシュへのデータ投入を行うかどうか。キャッシュへのデータ投入を完全にスキップするには、`--no-populate-cache` フラグまたは `DBT_POPULATE_CACHE: False` を使用します。ただし、これはキャッシュを無効化するものではありません。キャッシュ参照が失敗した場合はクエリが実行され、その後キャッシュが更新されます。
- `CACHE_SELECTED_ONLY` (デフォルト: `False`): キャッシュへのデータ投入を、現在の実行で選択されたリソースのみに制限するかどうか。これにより、大規模プロジェクトの小さなサブセットを実行する際に、事前キャッシュの利点を維持しながら、大幅な速度向上を実現できます。

たとえば、データベース メタデータやイントロスペクト クエリを必要としないモデルをすばやくコンパイルするには、次のようにします:

```text

dbt --no-populate-cache compile --select my_model_name

```

または、専用のスキーマに具体化される Salesforce モデルの開発に重点を置きながら速度とパフォーマンスを向上させるには、それらのモデルを選択し、`cache-selected-only` フラグを渡すこともできます:

```text

dbt --cache-selected-only run --select salesforce

```

### リレーショナルキャッシュイベントのログ記録

import LogLevel from '/snippets.ja/_log-relational-cache.md';

<LogLevel
event="relational cache"
/>
