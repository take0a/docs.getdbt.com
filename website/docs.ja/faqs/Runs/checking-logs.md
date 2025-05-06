---
title: dbt が実行している SQL を確認するにはどうすればよいでしょうか?
description: "ログを確認して、dbt が実行している SQL を確認します。"
sidebar_label: 'dbt が実行する SQL を確認する'
id: checking-logs

---

dbt が実行している SQL を確認するには、以下を参照してください。

* dbt Cloud:
  * 実行出力内でモデル名をクリックし、「詳細」を選択します。
* dbt Core:
  * コンパイルされた `select` ステートメントの `target/compiled/` ディレクトリ
  * コンパイルされた `create` ステートメントの `target/run/` ディレクトリ
  * 詳細ログの `logs/dbt.log` ファイル。
