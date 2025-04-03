---
title: Python のスタイル
id: 3-how-we-style-our-python
---

## Python ツール

- 🐍 Python には、フォーマットとリンティングのためのより成熟した堅牢なエコシステムがあります (100 万通りの異なる方言がないことが役立っています)。これらのツールを使用して、好みのスタイルでコードをフォーマットおよびリンティングすることをお勧めします。

- 🛠️ 現在の推奨事項は次のとおりです

  - [black](https://pypi.org/project/black/) formatter
  - [ruff](https://pypi.org/project/ruff/) linter

  :::info
  ☁️ dbt Cloud には、SQL を自動的に lint してフォーマットする [black formatter が組み込まれています](https://docs.getdbt.com/docs/cloud/dbt-cloud-ide/lint-format)。ダウンロードや設定は一切不要で、Python モデルで「Format」をクリックするだけで準備完了です。
  :::

## Pythonの例

```python
import pandas as pd


def model(dbt, session):
    # set length of time considered a churn
    pd.Timedelta(days=2)

    dbt.config(enabled=False, materialized="table", packages=["pandas==1.5.2"])

    orders_relation = dbt.ref("stg_orders")

    # converting a DuckDB Python Relation into a pandas DataFrame
    orders_df = orders_relation.df()

    orders_df.sort_values(by="ordered_at", inplace=True)
    orders_df["previous_order_at"] = orders_df.groupby("customer_id")[
        "ordered_at"
    ].shift(1)
    orders_df["next_order_at"] = orders_df.groupby("customer_id")["ordered_at"].shift(
        -1
    )
    return orders_df
```
