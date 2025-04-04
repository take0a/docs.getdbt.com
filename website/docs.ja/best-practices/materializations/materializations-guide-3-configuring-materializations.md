---
title: "マテリアライゼーションの設定"
id: materializations-guide-3-configuring-materializations
slug: 3-configuring-materializations
description: Read this guide to understand how to configure materializations in dbt.
displayText: Materializations best practices
hoverSnippet: Read this guide to understand how to configure materializations in dbt.
---

import Tabs from '@theme/Tabs';
import TabItem from '@theme/TabItem';

## マテリアライゼーションの設定

どのマテリアライゼーションを選択するかは、dbt で他の構成を設定するのと同じくらい簡単です。まず、個々のモデルのマテリアライゼーションを選択する方法を見て、次にモデルのフォルダー全体のマテリアライゼーションを設定するより強力な方法を見ていきます。

### テーブルとビューの構成

テーブルとビューを使用してマテリアライゼーションを開始する方法を見てみましょう:

- ⚙️ **Jinja `config` ブロック** を使用して、**`materialized` 引数** を渡すことで、個々のモデルのマテリアライゼーションを構成できます。これにより、dbt に使用するマテリアライゼーションが指示されます。
- 🚰 実行される内容の基本的な詳細は、[使用している **アダプタ**](/docs/supported-data-platforms) によって異なりますが、最終結果は同等になります。
- 😌 これは、dbt の多くの価値ある側面の 1 つです。dbt では、**宣言型** アプローチを使用できるため、コードで必要な _結果_ を、それを達成するための _特定の手順_ ではなく指定できます (コンピューター サイエンス的に言えば、後者は _命令型_ アプローチです 🤓)。
- 🔍 以下のケースでは、SQL **ビュー** を作成し、それを **1 行のコード** で **宣言** することができます。現時点では、Python モデルは [ビューとしてのマテリアライズをサポートしていない](https://docs.getdbt.com/docs/build/materializations#python-materializations) ことに注意してください。

```sql
    {{
        config(
            materialized='view'
        )
    }}

    select ...
```


:::info
🐍 **すべてのアダプタがまだ Python をサポートしているわけではありません**。Python モデルの作成に時間を費やす前に、[必ずこちらのドキュメント](/docs/build/python-models#specific-data-platforms)を確認してください。
:::

- モデルを `table` として実現するように構成するのは簡単で、SQL モデルと Python モデルの両方で可能です。

<Tabs>
<TabItem value="sql" label="SQL">

```sql
{{
    config(
        materialized='table'
    )
}}

select ...
```

</TabItem>
<TabItem value="python" label="Python">

```python
def model(dbt, session):

    dbt.config(materialized="table")

    # model logic

    return model_df
```

</TabItem>
</Tabs>

ぜひ、これらのいくつかを試してみてください。
