---
title: "JSONアーティファクト"
id: "json-artifacts"
sidebar: "JSONアーティファクト"
---

### JSON アーティファクトの書き込み

`WRITE_JSON` 設定は、dbt が [JSON アーティファクト](/reference/artifacts/dbt-artifacts) (例: `manifest.json`、`run_results.json`) を `target/` ディレクトリに書き込むかどうかを決定します。JSON シリアル化は時間がかかる場合があり、このフラグをオフにすると dbt の呼び出しが高速化される可能性があります。あるいは、この設定を無効にして dbt 操作を実行し、以前の実行ステップのアーティファクトが上書きされないようにすることもできます。

<File name='Usage'>

```text
dbt run --no-write-json 
```

</File>


### ターゲットパス

デフォルトでは、dbt は JSON アーティファクトとコンパイル済み SQL ファイルを `target/` というディレクトリに書き込みます。このディレクトリは、アクティブプロジェクトの `dbt_project.yml` を基準とした相対パスです。

他のグローバル設定と同様に、CLI オプション (`--target-path`) または環境変数 (`DBT_TARGET_PATH`) を使用して、環境や呼び出しに合わせてこれらの値を上書きできます。

