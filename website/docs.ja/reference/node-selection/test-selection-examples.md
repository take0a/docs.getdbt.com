---
title: "test での選択例"
---

import IndirSelect from '/snippets.ja/_indirect-selection-definitions.md';

テストの選択は、他のリソースの選択とは少し異なる仕組みになっています。これにより、以下の操作が非常に簡単になります:
* 特定のモデルでテストを実行する
* サブディレクトリ内のすべてのモデルでテストを実行する
* モデルの上流 / 下流にあるすべてのモデルでテストを実行する など。

他のリソースタイプと同様に、テストは、名前、プロパティ、タグなどの属性のいずれかを取得するメソッドや演算子によって**直接**選択できます。

他のリソースタイプとは異なり、テストは**間接**的に選択することもできます。選択メソッドまたは演算子にテストの親が含まれている場合、そのテストも選択されます。詳細については、[下記を参照](#indirect-selection)してください。

テストの選択は強力ですが、扱いが難しい場合もあることを私たちは認識しています。そのため、以下に多くの例を挙げました:

### 直接選択

汎用テストのみ実行:


  ```bash
    dbt test --select "test_type:generic"
  ```

単一テストのみを実行:


  ```bash
    dbt test --select "test_type:singular"
  ```

どちらの場合も、`test_type` はテスト自体のプロパティをチェックします。これらは「直接的な」テスト選択の形式です。

### 間接選択

<IndirSelect features={'/snippets.ja/indirect-selection-definitions.md'}/>

<!--tabs for eager mode, cautious mode, empty, and buildable mode -->
<!--Tabs for 1.5+ -->

### 間接選択の例

これらの手法を視覚的に理解するために、`model_a`、`model_b`、`model_c` とそれに関連するデータテストがあると仮定します。以下は、`dbt build` を様々な間接選択モードで実行した場合に実行されるテストを示しています。

<DocCarousel slidesPerView={1}>

<Lightbox src src="/img/docs/reference/indirect-selection-dbt-build.png" width="85%" title="dbt build" />

<Lightbox src src="/img/docs/reference/indirect-selection-eager.png" width="85%" title="Eager (default)"/>

<Lightbox src src="/img/docs/reference/indirect-selection-buildable.png" width="85%" title="Buildable"/>

<Lightbox src src="/img/docs/reference/indirect-selection-cautious.png" width="85%" title="Cautious"/>

<Lightbox src src="/img/docs/reference/indirect-selection-empty.png" width="85%" title="Empty"/>

</DocCarousel>

<Tabs queryString="indirect-selection-mode">
<TabItem value="eager" label="Eager mode (default)">

この例では、ビルド プロセス中に、選択した「orders」モデルまたはその依存モデルに依存するすべてのテストが、他のモデルにも依存している場合でも実行されます。
 
```shell
dbt test --select "orders"
dbt build --select "orders"
```

</TabItem>

<TabItem value="buildable" label="Buildable mode">

この例では、dbt は、選択したノード (またはその祖先) 内の「順序」を参照するテストを実行します。


```shell
dbt test --select "orders" --indirect-selection=buildable
dbt build --select "orders" --indirect-selection=buildable
```

</TabItem>

<TabItem value="cautious" label="Cautious mode">

この例では、「orders」モデルに排他的に依存するテストのみが実行されます:

```shell
dbt test --select "orders" --indirect-selection=cautious
dbt build --select "orders" --indirect-selection=cautious

```

</TabItem>

<TabItem value="empty" label="Empty mode">

このモードでは、選択したノードに直接接続されているかどうかに関係なく、テストは実行されません。

```shell

dbt test --select "orders" --indirect-selection=empty
dbt build --select "orders" --indirect-selection=empty

```

</TabItem>

</Tabs>

<!--End of tabs for eager mode, cautious mode, buildable mode, and empty mode -->

### test での選択構文の例

`indirect_selection` の設定は、[yaml セレクター](/reference/node-selection/yaml-selectors#indirect-selection) でも指定できます。

DAG の一部をビルドするために `--select` オプションを指定して `dbt run` を実行することに慣れている方であれば、以下の例は馴染みがあるはずです:


  ```bash
  # Run tests on a model (indirect selection)
  dbt test --select "customers"
  
  # Run tests on two or more specific models (indirect selection)
  dbt test --select "customers orders"

  # Run tests on all models in the models/staging/jaffle_shop directory (indirect selection)
  dbt test --select "staging.jaffle_shop"

  # Run tests downstream of a model (note this will select those tests directly!)
  dbt test --select "stg_customers+"

  # Run tests upstream of a model (indirect selection)
  dbt test --select "+stg_customers"

  # Run tests on all models with a particular tag (direct + indirect)
  dbt test --select "tag:my_model_tag"

  # Run tests on all models with a particular materialization (indirect selection)
  dbt test --select "config.materialized:table"

  ```

同じ原則は、他のリソースタイプに定義されたテストにも適用できます。この場合、`source:` 選択メソッドを使用して、特定のソースに定義されたすべてのテストを実行します:

  ```bash
  # tests on all sources

  dbt test --select "source:*"

  # tests on one source
  dbt test --select "source:jaffle_shop"
  
  # tests on two or more specific sources
   dbt test --select "source:jaffle_shop source:raffle_bakery"

  # tests on one source table
  dbt test --select "source:jaffle_shop.customers"

  # tests on everything _except_ sources
  dbt test --exclude "source:*"
  ```

 ### より複雑な選択

直接的な選択と間接的な選択を組み合わせることで、同じ結果を得る方法は複数あります。例えば、「payments」というモデルに依存する「assert_total_payment_amount_is_positive」というデータテストがあるとします。以下のすべての方法で、このテストを個別に選択して実行できます:


  ```bash

  dbt test --select "assert_total_payment_amount_is_positive" # directly select the test by name
  dbt test --select "payments,test_type:singular" # indirect selection, v1.2

  ```


リソースグループに共通するプロパティを選択できる限り、間接選択によってそれらのリソースに対してもすべてのテストを実行できます。上記の例では、すべてのテーブルマテリアライズドモデルをテストできることがわかりました。この原則は他のリソースタイプにも拡張できます:


  ```bash
  # Run tests on all models with a particular materialization
  dbt test --select "config.materialized:table"

  # Run tests on all seeds, which use the 'seed' materialization
  dbt test --select "config.materialized:seed"

  # Run tests on all snapshots, which use the 'snapshot' materialization
  dbt test --select "config.materialized:snapshot"

  ```

この機能は dbt の将来のバージョンで変更される可能性があることに注意してください。

### タグ付き列でテストを実行する

列「order_id」には「my_column_tag」タグが付けられているため、テスト自体にも「my_column_tag」タグが付与されます。そのため、これは直接選択の例です。

<File name='models/<filename>.yml'>

```yml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tags: [my_column_tag]
        tests:
          - unique

```

</File>


  ```bash
  dbt test --select "tag:my_column_tag"

  ```

現在、テストは列、ソース、ソーステーブルに適用されたタグを「継承」します。モデル、シード、スナップショットに適用されたタグは継承しません。タグは親を選択するため、これらのテストは間接的に選択される可能性が高いです。これは微妙な違いであり、dbtの将来のバージョンで変更される可能性があります。

### タグ付きテストのみ実行

これは直接選択のさらに明確な例です。テスト自体に `my_test_tag` というタグが付けられており、それに応じて選択されています。

<File name='models/<filename>.yml'>

```yml
version: 2

models:
  - name: orders
    columns:
      - name: order_id
        tests:
          - unique:
              tags: [my_test_tag]

```

</File>


  ```bash
  dbt test --select "tag:my_test_tag"

  ```
