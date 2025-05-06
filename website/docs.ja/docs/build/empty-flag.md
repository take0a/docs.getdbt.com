---
title: empty フラグについて"
description: "empty フラグを使用してコードをテストし、データを入力せずにテーブルを構築します。"
sidebar_label: "The empty flag"
pagination_next: "docs/build/sample-flag"
pagination_prev: null
---

# `--empty`フラグについて

:::note

`--empty` フラグは現在 Python モデルでは使用できません。このフラグを Python モデルで使用した場合、無視されます。

:::

dbt 開発中に、データウェアハウスでモデル全体を構築するという時間のかかるコストをかけずに、モデルがセマンティックに正しいことを検証したい場合があります。[`run`](/reference/commands/run) コマンドと [`build`](/reference/commands/build) コマンドは、スキーマのみのドライランを構築するための `--empty` フラグをサポートしています。`--empty` フラグは、参照とソースを 0 行に制限します。dbt はターゲット データウェアハウスに対してモデル SQL を実行しますが、入力データの読み取りにかかるコストを回避します。これにより、依存関係が検証され、モデルが適切に構築されることが保証されます。

### 例

開発環境でスキーマのみを構築しながら、プロジェクト内のすべてのモデルを実行します:

```
dbt run --empty
```

特定のモデルを実行します:

```
dbt run --select path/to/your_model --empty
```

dbt は SQL を構築して実行し、データ ウェアハウスに空のスキーマを作成します。

