---
title: "高度なデータモデリング"
description: "Learn about advanced topics for dbt Semantic Layer and MetricFlow, such as modeling workflows and more."
pagination_prev: null
---

dbt セマンティックレイヤーと MetricFlow は、dbt プロジェクトでメトリクスとセマンティックモデルを定義できる強力なツールです。

このセクションでは、データモデリングワークフローなど、dbt セマンティックレイヤーと MetricFlow の高度なトピックについて説明します。
<!--
- [Fill null values for simple and derived or ratio metrics](/docs/build/fill-nulls-advanced) &mdash; Use `fill_nulls_with` to set null metric values to zero, ensuring numeric values for every data row, even with derived metrics.
-->

<div className="grid--2-col">

<Card
    title="メトリックのnull値を入力する"
    body="<code>fill_nulls_with</code> を使用して null メトリック値をゼロに設定し、すべてのデータ行に数値が確保されるようにします。"
    link="/docs/build/fill-nulls-advanced"
    icon="dbt-bit"/>

<Card
    title="メトリックフィルターを使用したディメンションとしてのメトリック"
    body="メトリックをディメンションとしてメトリック フィルターに追加して、より複雑なメトリックを作成し、より多くの分析情報を得ることができます。"
    link="/docs/build/ref-metrics-in-filters"
    icon="dbt-bit"/>

</div>
