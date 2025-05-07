---
resource_types: [models]
datatype: deprecation_date
required: no
---

<File name='models/<schema>.yml'>

```yml
models:
  - name: my_model
    description: deprecated
    deprecation_date: 1999-01-01 00:00:00.00+00:00
```
</File>

<File name='models/<schema>.yml'>

```yml
version: 2
models:
  - name: my_model
    description: deprecating in the future
    deprecation_date: 2999-01-01 00:00:00.00+00:00
```

</File>

## 定義

モデルの廃止日は日付形式で表され、オプションでタイムゾーンオフセットも指定できます。サポートされているRFC 3339形式は次のとおりです:
- `YYYY-MM-DD hh:mm:ss.sss±hh:mm`
- `YYYY-MM-DD hh:mm:ss.sss`
- `YYYY-MM-DD`

`deprecation_date` に UTC からのオフセットが含まれていない場合、dbt 実行環境のシステム タイム ゾーンにあると解釈されます。

## 説明

### 目的

dbt モデルに `deprecation_date` を宣言することで、長期的なサポートとメンテナンスの計画とタイムラインを伝え、変更管理を容易にするメカニズムが提供されます。

`deprecation_date` の設定は、[モデルバージョン](/docs/collaborate/govern/model-versions) などの他の [モデルガバナンス](/docs/collaborate/govern/about-model-governance) 機能と連携して機能しますが、それらとは独立して使用することもできます。

### 警告メッセージ

プロジェクトが廃止予定のモデル、または廃止予定日を過ぎたモデルを参照している場合、警告が生成されます。バージョン管理されたモデルで、新しいバージョンが利用可能な場合は、警告にその旨が表示されます。プロデューサーからコンシューマーまで、チーム間のコミュニケーションが促進されるこの仕組みは、モデルバージョンに関する dbt の組み込み機能を使用して移行を円滑に進めるメリットです。

さらに、[`WARN_ERROR_OPTIONS`](/reference/global-configs/warnings) を使用すると、ユーザーはこれらの警告を実際のランタイムエラーに昇格させることができます。

| Warning                        | Scenario                                           | Affected projects      |
|--------------------------------|----------------------------------------------------|------------------------|
|        `DeprecatedModel`       | 非推奨のモデルを定義するプロジェクトの解析  | Producer               |
| `DeprecatedReference`          | 廃止日が過ぎたモデルを参照する   | Producer and consumers |
| `UpcomingReferenceDeprecation` | 将来の廃止予定日を持つモデルを参照する | Producer and consumers |

**例**

`UpcomingReferenceDeprecation` 警告の出力例:

```
$ dbt parse
15:48:14  Running with dbt=1.6.0
15:48:14  Registered adapter: postgres=1.6.0
15:48:14  [WARNING]: While compiling 'my_model_ref': Found a reference to my_model, which is slated for deprecation on '2038-01-19T03:14:07-00:00'.
```

### 選択構文

`deprecation_date` には特定の [ノード選択構文](/reference/node-selection/syntax) はありません。[プログラムによる呼び出し](/reference/programmatic-invocations) は、非推奨モデルを識別する方法の 1 つです（[dbt リスト](/reference/commands/list) と併用することもできます）。例: `dbt -q ls --output json --output-keys database schema alias deprecation_date`。

### 非推奨プロセス

非推奨モデルのビルド関連のコンピューティングおよびストレージコストを削減するには、追加の手順が必要です。

非推奨モデルは、[無効化](/reference/resource-configs/enabled)または削除されるまで、プロデューサーによって引き続きビルドされ、コンシューマーによって選択されます。

モデルが削除されてもリレーションが自動的に削除されないのと同様に、dbt は非推奨モデルのリレーションを削除しません。

[こちら](https://discourse.getdbt.com/t/faq-cleaning-up-removed-models-from-your-production-schema/113) や [こちら](https://discourse.getdbt.com/t/clean-your-warehouse-of-old-and-deprecated-models/1547) と同様の戦略を使用して、非推奨となり使用されなくなったリレーションを削除できます。

### BigQuery のテーブルの有効期限

dbt-bigquery は、[`hours_to_expiration`](/reference/resource-configs/bigquery-configs#controlling-table-expiration) を設定できます。これは BigQuery 内で `expiration_timestamp` に変換されます。

dbt は `deprecation_date` と `hours_to_expiration` を自動的に同期しませんが、ユーザーは何らかの方法でこれらを調整することができます（例えば、モデルの有効期限を `deprecation_date` の 48 時間後に設定するなど）。BigQuery 内の有効期限切れのテーブルは削除され、ストレージが再利用されます。
