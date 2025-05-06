---
title: テスト失敗のしきい値を設定できますか?
description: "構成を使用してテストのカスタム失敗しきい値を設定する"
sidebar_label: 'テストで失敗しきい値を設定する方法'
id: custom-test-thresholds

---

`error_if` および `warn_if` 構成を使用して、テストでカスタムの失敗しきい値を設定できます。詳細については、[リファレンス](/reference/resource-configs/severity) を参照してください。

以下の解決策もお試しください。

* [severity](/reference/resource-properties/data-tests#severity) を `warn` に設定する。または、以下の手順を実行してください。
* しきい値引数を受け入れる [カスタム汎用テスト](/best-practices/writing-custom-generic-tests) を作成する ([例](https://discourse.getdbt.com/t/creating-an-error-threshold-for-schema-tests/966))
