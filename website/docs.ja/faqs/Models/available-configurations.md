---
title: どのようなモデル構成がありますか?
description: "モデル構成について学ぶ"
sidebar_label: 'モデル構成'
id: available-configurations
---
以下の設定も可能です。

* [タグ](/reference/resource-configs/tags) : 分類とグラフ選択を容易にします。
* [カスタム スキーマ](/reference/resource-properties/schema) : モデルを複数のスキーマに分割します。
* [エイリアス](/reference/resource-configs/alias) : <Term id="view" />/<Term id="table" /> の名前をファイル名と異なるものにする場合。
* [フック](/docs/build/hooks-operations) と呼ばれる、モデルの開始時または終了時に実行する SQL スニペット。
* パフォーマンス向上のためのウェアハウス固有の設定 (例: Redshift の `sort` キーと `dist` キー、BigQuery の `partitions` キー)

詳細については、[モデル設定](/reference/model-configs) のドキュメントをご覧ください。
