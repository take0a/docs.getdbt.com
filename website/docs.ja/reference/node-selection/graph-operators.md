---
title: "Graph 演算子"
---

### 「プラス」演算子
`+` 演算子は、リソースの祖先（上流の依存関係）または子孫（下流の依存関係）を含むように選択範囲を拡張します。この演算子は、個々のモデル、タグ、その他のリソースに使用できます。

- モデル/リソースの後に配置する場合 - リソース自体とそのすべての子孫（下流の依存関係）が含まれます。
- モデル/リソースの前に配置する場合 - リソース自体とそのすべての祖先（上流の依存関係）が含まれます。
- モデル/リソースの両側に配置する場合 - リソース自体、そのすべての祖先、およびすべての子孫が含まれます。

```bash
dbt run --select "my_model+"         # select my_model and all descendants
dbt run --select "+my_model"         # select my_model and all ancestors
dbt run --select "+my_model+"        # select my_model, and all of its ancestors and descendants
```

セレクターと組み合わせて使用​​することで、コマンドの適用範囲をより限定的に指定できます。また、[`--exclude`](/reference/node-selection/exclude) フラグと組み合わせることで、コマンドに含める内容をさらに細かく制御できます。

### 「nプラス」演算子

ステップスルーするエッジの数を数値化することで、「+」演算子の動作を調整できます。


  ```bash
dbt run --select "my_model+1"        # select my_model and its first-degree descendants
dbt run --select "2+my_model"        # select my_model, its first-degree ancestors ("parents"), and its second-degree ancestors ("grandparents")
dbt run --select "3+my_model+4"      # select my_model, its ancestors up to the 3rd degree, and its descendants down to the 4th degree
  ```


### 「at」演算子
`@` 演算子は `+` に似ていますが、_選択したモデルのすべての子孫のすべての祖先_ も含めます。これは、モデルとそのすべての子孫を構築したいものの、それらの子孫の祖先がまだスキーマに存在しない可能性がある継続的インテグレーション環境で役立ちます。`@` 演算子（モデル名の先頭にのみ配置可能）は、指定されたモデルのすべての子孫を正常に構築するために必要な数の祖先（「親」、「祖父母」など）を選択します。

セレクタ `@snowplow_web_page_context` は、下の図に示す 3 つのモデルすべてを構築します。

<Lightbox src="/img/docs/running-a-dbt-project/command-line-interface/1643e30-Screen_Shot_2019-03-11_at_7.18.20_PM.png" title="@snowplow_web_page_context will select all of the models shown here"/>

```bash
dbt run --select "@my_model"         # select my_model, its descendants, and the ancestors of its descendants
```
