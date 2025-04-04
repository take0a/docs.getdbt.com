---
title: "ビルドの検証"
id: materializations-guide-6-examining-builds
slug: 6-examining-builds
description: Read this guide to understand how to examine your builds in dbt.
displayText: Materializations best practices
hoverSnippet: Read this guide to understand how to examine your builds in dbt.
---

## ビルドの検証

- ⌚ dbt は、**各モデルの構築にかかった時間**、開始日時、終了日時、完了ステータス (エラー、警告、または成功)、実現タイプ、その他多くの情報を追跡します。
- 🖼️ この情報は、dbt が **アーティファクト** と呼ぶいくつかのファイルに保存されます。
- 📊 アーティファクトには JSON 形式の大量の情報が含まれているため、読み取るのは簡単ではありませんが、**dbt Cloud** は最も有用な情報を整理された **視覚化** 形式でパッケージ化します。
- ☁️ クラウドを使用していない場合でも、**dbt Core CLI** の出力を使用して実行を把握できます。

### モデルタイミング

ここで、dbt Cloud のモデルタイミング視覚化が非常に役立ちます。dbt Cloud でモデルを実行するための [ジョブ](/guides/bigquery) を設定した場合、[モデルタイミング] タブを使用して、最も実行時間が長いモデルを特定できます。

![dbt Cloud's Model Timing diagram](/img/best-practices/materializations/model-timing-diagram.png)

- 🧵 このビューでは、**スレッドにマッピングされた** (最大 64 スレッド、現在は 4 つのスレッドで実行されているため、4 つのトラックがあります) を時間の経過とともに確認できます。**各スレッドは高速道路の車線** と考えることができます。
- ⌛ 上記から、`customer_status_histories` が **圧倒的に最も時間がかかっている** ことがわかります。そのため、先に進んで **増分処理にする** ことをお勧めします。

dbt Cloud を使用していなくても大丈夫です。すぐに使える派手な視覚化は得られませんが、dbt Core CLI からの出力を使用してモデル時間をチェックできます。これは、その出力に慣れる絶好の機会です。

### dbt Core CLI の出力

`build`、`test`、`run` など、dbt を実行したことがある場合は、以下のような出力を見たことがあるでしょう。これをどのように読み取るかを詳しく見てみましょう。

![CLI output from a dbt build command](/img/best-practices/materializations/dbt-build-output.png)

- モデルごとに 2 つのエントリがあります。モデルのビルドの **開始** と **完了** です。これには、モデルの実行にかかった **時間** が含まれます。モデルの **タイプ** も含まれます。例:

```shell
20:24:51  5 of 10 START sql view model main.stg_products ......... [RUN]
20:24:51  5 of 10 OK created sql view model main.stg_products .... [OK in 0.13s]
```

- 5️⃣  **両方の行** では、`stg_products` モデルが構築中の 10 個のオブジェクトのうち 5 番目であること、開始されたタイムスタンプ、SQL で定義されていること (Python ではなく)、およびビューであることがわかります。
- 🆕  **最初の行** には、モデルが **開始** されたときのタイムスタンプが表示されます。
- ✅  **2 行目** には、**ステータス** (このモデルの実行中に他のモデルが開始および終了するスレッドがあるため) と **ビルド時間** (この場合は `OK`) を追加する **完了** エントリが表示されます。これは、非常に速い 0.13 秒です。ビューについて私たちが知っていることを考えると、予想外のことではありません。
- 🏎️  **ビューは通常 1 〜 2 秒もかかりませんが、** これらのツールではテーブルと増分モデルを注意深く監視する必要があります。

### dbt アーティファクトパッケージ

- 🎨  最後に、dbt 実行を調べる場合、dbt Core を使用している場合は、**派手なビジュアルなしで困ることはありません**。すぐに使用できる状態では設定されていませんが、プロジェクトをより深くイントロスペクトしたい場合は、[dbt Artifacts パッケージ](https://github.com/brooklyn-data/dbt_artifacts) を使用できます。
- 👩‍🎨  これにより、**プロジェクトのあらゆる側面を非常に詳細なレベルで視覚化**できるモデルが提供されます。
- ⌚  これを使用して、BI ツールで**独自のモデルタイミング視覚化**を作成したり、実現化戦略を監視するために必要なその他のレポートを作成したりできます。
