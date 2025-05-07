---
resource_types: [models]
datatype: list
required: no
keyword: governance, model version, model versioning, dbt model versioning
---

import VersionsCallout from '/snippets.ja/_model-version-callout.md';

<VersionsCallout />

<File name='models/<schema>.yml'>

```yml
version: 2

models:
  - name: model_name
    versions:
      - v: <version_identifier> # required
        defined_in: <file_name> # optional -- default is <model_name>_v<v>
        columns:
          # specify all columns, or include/exclude columns from the top-level model YAML definition
          - [include](/reference/resource-properties/versions#include): <include_value>
            [exclude](/reference/resource-properties/versions#include): <exclude_list>
          # specify additional columns
          - name: <column_name> # required
      - v: ...
    
    # optional
    [latest_version](/reference/resource-properties/latest_version): <version_identifier> 
```

</File>

モデルバージョンの命名規則は `<model_name>_v<v>` です。これは、dbt がモデルの定義（SQL または Python）を記述するファイルと、データベースにモデルをマテリアライズする際にデフォルトで使用されるエイリアスに適用されます。

### `v`

モデルのバージョンを表すバージョン識別子。この値は、数値（整数または浮動小数点数）または任意の文字列です。

バージョン識別子の値は、モデルのバージョンを相対的に順序付けるために使用されます。バージョン管理されたモデルで [`latest_version`](/reference/resource-properties/latest_version) が明示的に設定されていない場合、`version` 引数を指定せずにモデルへの `ref` 呼び出しを解決する際に、最も高いバージョン番号が最新バージョンとして使用されます。

一般的に、モデルにはシンプルな「メジャー バージョン管理」スキーム（`1`、`2`、`3` など）を使用することをお勧めします。各バージョンは、以前のバージョンからの互換性を破る変更を反映します。他のバージョン管理スキームも使用できます。dbt は、値がすべて数値でない場合、バージョン識別子をアルファベット順に並べ替えます。バージョン識別子に文字 `v` を含めないでください。dbt が自動的に行います。

複数のバージョンを持つモデルを実行するには、[`--select` フラグ](/reference/node-selection/syntax) を使用します。詳細と構文については、[モデルのバージョン](/docs/collaborate/govern/model-versions#run-a-model-with-multiple-versions) を参照してください。


### `defined_in`

モデルバージョンが定義されているモデルファイルの名前（ファイル拡張子（例：`.sql` または `.py`）を除く）。

`defined_in` が指定されていない場合、dbt はバージョン管理されたモデルの定義を `<model_name>_v<v>` という名前のモデルファイルで検索します。モデルの**最新**バージョンは、バージョンサフィックスのない `<model_name>` という名前のファイルで定義することもできます。モデルファイル名は、異なる名前でモデルのバージョン管理された実装を定義する場合でも、グローバルに一意である必要があります。

### `alias`

バージョン管理されたモデルのデフォルトの解決済み `alias` は `<model_name>_v<v>` です。このロジックは `generate_alias_name` マクロにエンコードされています。

このデフォルトは、次の 2 つの方法で上書きできます。
- バージョン管理されたモデルの定義またはバージョン管理された YAML 内でカスタム `alias` を設定する
- dbt の `generate_alias_name` マクロを上書きし、`node.version` に基づいて異なる動作を使用する

詳細については、[「カスタム エイリアス」](https://docs.getdbt.com/docs/build/custom-aliases) を参照してください。

モデルの `defined_in` の値と `alias` 設定は、慣例による場合を除き、連携しないことに注意してください。この 2 つは独立して宣言および決定されます。

### `include`

モデルの最上位レベルの `columns` プロパティで定義されている列のうち、バージョン管理されたモデル実装に含めるか除外するかを指定します。

`include` は次のいずれかです。
- 含める列名のリスト
- `'*'` または `'all'`。これは、最上位レベルの `columns` プロパティの **すべての** 列をバージョン管理されたモデルに含めることを示します。

`exclude` は除外する列名のリストです。`include` が `'*'` または `'all'` のいずれかに設定されている場合にのみ宣言できます。

バージョン管理されたモデルの `columns` リストには、最大で 1 つの `include/exclude` 要素を含めることができます。

バージョンの `columns` リスト内に追加の列を宣言できます。バージョン固有の列の `name` が最上位レベルから含められる列と一致する場合、そのバージョンではバージョン固有のエントリがその列をオーバーライドします。

<File name='models/<schema>.yml'>

```yml
version: 2

models:
  
  # top-level model properties
  - name: <model_name>
    [columns](/reference/resource-properties/columns):
      - name: <column_name> # required
    
    # versions of this model
    [versions](/reference/resource-properties/versions):
      - v: <version_identifier> # required
        columns:
          - include: '*' | 'all' | [<column_name>, ...]
            exclude:
              - <column_name>
              - ... # declare additional column names to exclude
          
          # declare more columns -- can be overrides from top-level, or in addition
          - name: <column_name>
            ...

```

</File>

デフォルトでは、`include` は "all"、`exclude` は空のリストです。これにより、ベースモデルのすべての列がバージョン管理モデルに含められます。

#### 例

<File name='models/customers.yml'>

```yml
models:
  - name: customers
    columns:
      - name: customer_id
        description: Unique identifier for this table
        data_type: text
        constraints:
          - type: not_null
        tests:
          - unique
      - name: customer_country
        data_type: text
        description: "Country where the customer currently lives"
      - name: first_purchase_date
        data_type: date
    
    versions:
      - v: 4
      
      - v: 3
        columns:
          - include: "*"
          - name: customer_country
            data_type: text
            description: "Country where the customer first lived at time of first purchase"
      
      - v: 2
        columns:
          - include: "*"
            exclude:
              - customer_country
      
      - v: 1
        columns:
          - include: []
          - name: id
            data_type: int
```

</File>

`v4` は `columns` を指定していないため、トップレベルの `columns` をすべて含めます。

その他のバージョンでは、トップレベルのプロパティから変更が宣言されています。
- `v3` はすべての列を含めますが、`customer_country` 列を異なる `description` で再実装します。
- `v2` は `customer_country` を除くすべての列を含めます。
- `v1` はトップレベルの `columns` を *一切* 含めません。代わりに、`id` という名前の単一の整数列のみを宣言します。


### 推奨事項
- モデルのバージョンとエイリアスには、一貫した命名規則に従ってください。
- `defined_in` と `alias` は、正当な理由がある場合のみ使用してください。
- 常にモデルの最新バージョンを指すビューを作成してください。`on-run-end` フックを使用することで、プロジェクト内のすべてのバージョン管理モデルに対してこれを自動化できます。詳細については、["モデルのバージョン"](/docs/collaborate/govern/model-versions#configuring-database-location-with-alias) の完全なドキュメントをご覧ください。

### 破壊的変更の検出

Slim CI で `state:modified` 選択メソッドを使用すると、dbt はバージョン管理されたモデルコントラクトへの変更を検出し、下流のコンシューマーにとって破壊的となる可能性のある変更があった場合はエラーを発生します。

import BreakingChanges from '/snippets/_versions-contracts.md';

<BreakingChanges 
value="Changing unversioned, contracted models."
value2="dbt also warns if a model has or had a contract but isn't versioned."
/>

<Tabs>

<TabItem value="unversioned" label="バージョン管理されていないモデルのメッセージの例">

```
  Breaking Change to Unversioned Contract for contracted_model (models/contracted_models/contracted_model.sql)
  While comparing to previous project state, dbt detected a breaking change to an unversioned model.
    - Contract enforcement was removed: Previously, this model's configuration included contract: {enforced: true}. It is no longer configured to enforce its contract, and this is a breaking change.
    - Columns were removed:
      - color
      - date_day
    - Enforced column level constraints were removed:
      - id (ConstraintType.not_null)
      - id (ConstraintType.primary_key)
    - Enforced model level constraints were removed:
      - ConstraintType.check -> ['id']
    - Materialization changed with enforced constraints:
      - table -> view
```
</TabItem>

<TabItem value="versioned" label="バージョン管理されたモデルのメッセージの例">

```
Breaking Change to Contract Error in model sometable (models/sometable.sql)
  While comparing to previous project state, dbt detected a breaking change to an enforced contract.

  The contract's enforcement has been disabled.

  Columns were removed:
   - order_name

  Columns with data_type changes:
   - order_id (number -> int)

  Consider making an additive (non-breaking) change instead, if possible.
  Otherwise, create a new model version: https://docs.getdbt.com/docs/collaborate/govern/model-versions
```

</TabItem>


</Tabs>

追加的な変更は、**破壊的変更とはみなされません**。
- 縮小モデルに新しい列を追加する
- 縮小モデル内の既存の列に新しい「制約」を追加する
