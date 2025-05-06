---
title: dbt ではどのようなマテリアライゼーションが利用できますか?
description: "dbtは4つのマテリアライゼーションを使用する"
sidebar_label: '可能なマテリアライゼーション'
id: available-materializations
---

dbt には、5 つの <Term id="materialization">マテリアライゼーション</Term>（`view`、`table`、`incremental`、`ephemeral`、`materialized_view`）が付属しています。
各オプションの詳細については、[マテリアライゼーション](/docs/build/materializations) に関するドキュメントをご覧ください。

必要に応じて独自の [カスタム マテリアライゼーション](/guides/create-new-materializations) を作成することもできますが、これは dbt の高度な機能です。
