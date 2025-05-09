---
title: "バージョンの互換性を確認する"
id: "version-compatibility"
sidebar: "バージョン互換性"
---

dbt Core の開発開始から数年間は、互換性を破る変更がより頻繁に発生していました。そのため、[dbt のバージョン要件](/reference/project-configs/require-dbt-version) を設定することを推奨していました。特に、新しい機能や将来のバージョンの dbt Core で互換性がなくなる可能性のある機能を使用する場合は、この設定が重要です。デフォルトでは、互換性のない dbt バージョンでプロジェクトを実行すると、dbt はエラーを生成します。

`VERSION_CHECK` 設定を使用すると、このチェックを無効にしてエラーメッセージを抑制できます。

```
dbt --no-version-check run
Running with dbt=1.0.0
Found 13 models, 2 tests, 1 archives, 0 analyses, 204 macros, 2 operations....
```

:::note dbt Cloud release tracks
<Snippet path="_config-dbt-version-check" />

:::
