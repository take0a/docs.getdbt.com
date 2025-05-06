---
title: 最新スナップショットからテーブルを除外するにはどうすればよいですか?
description: "最新スナップショットからテーブルを除外するには null を使用します"
sidebar_label: '最新スナップショットからテーブルを除外する'
id: exclude-table-from-freshness

---

データソース内の一部のテーブルは、更新頻度が低い場合があります。ソースレベルで `freshness` プロパティを設定している場合、この <Term id="table" /> はチェックに失敗する可能性があります。

この問題を回避するには、テーブルの freshness を null (`freshness: null`) に設定して、特定のテーブルの freshness を「設定解除」します:

<File name='models/<filename>.yml'>

```yaml

version: 2

sources:
  - name: jaffle_shop
    database: raw

    freshness:
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}

    loaded_at_field: _etl_loaded_at

    tables:
      - name: orders
      - name: product_skus
        freshness: null # do not check freshness for this table
```

</File>
