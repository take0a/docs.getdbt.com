---
title: "YAML Selectors"
---

リソースセレクターをYAMLで記述し、わかりやすい名前で保存し、`--selector`フラグを使用して参照します。
セレクターをトップレベルの`selectors.yml`ファイルに記録することで、以下のメリットが得られます。

* **可読性:** 複雑な選択基準は、辞書と配列で構成されます。
* **バージョン管理:** セレクター定義は、dbtプロジェクトと同じgitリポジトリに保存されます。
* **再利用性:** セレクターは複数のジョブ定義で参照でき、定義は拡張可能です（YAMLアンカーを使用）。

セレクターは、トップレベルの`selectors.yml`ファイルに記述されます。各セレクターには`name`と`definition`が必要です。また、オプションで`description`と[`default`フラグ](#default)を定義できます。

<File name='selectors.yml'>

```yml
selectors:
  - name: nodes_to_joy
    definition: ...
  - name: nodes_to_a_grecian_urn
    description: Attic shape with a fair attitude
    default: true
    definition: ...
```
</File>

## 定義

各 `definition` は、1 つ以上の引数で構成されます。引数は以下のいずれかになります。
* **CLI スタイル:** 文字列。CLI スタイルの引数を表します。
* **Key-Value:** ペア。`method: value` 形式
* **Full YAML:** `method`、`value`、演算子に相当するキーワード、`exclude` のサポートを含む、完全に指定された辞書。

複数の引数を整理するには、`union` および `intersection` 演算子に相当するキーワードを使用します。

### CLI-style

```yml
definition:
  'tag:nightly'
```

この単純な構文は、`+`、`@`、および `*` [graph](/reference/node-selection/graph-operators) 演算子の使用をサポートしていますが、[set](/reference/node-selection/set-operators) 演算子または `exclude` はサポートしていません。

### Key-value

```yml
definition:
  tag: nightly
```

この単純な構文は、[graph](/reference/node-selection/graph-operators) 演算子や [set](/reference/node-selection/set-operators) 演算子、あるいは `exclude` をサポートしていません。

### Full YAML

これは最も包括的な構文であり、[graph](/reference/node-selection/graph-operators) および [set](/reference/node-selection/set-operators) 演算子の同等のキーワードを含めることができます。

使用可能なメソッドのリストについては、[methods](/reference/node-selection/methods) を参照してください。

```yml
definition:
  method: tag
  value: nightly

  # Optional keywords map to the `+` and `@` graph operators:

  children: true | false
  parents: true | false

  children_depth: 1    # if children: true, degrees to include
  parents_depth: 1     # if parents: true, degrees to include

  childrens_parents: true | false     # @ operator

  indirect_selection: eager | cautious | buildable | empty # include all tests selected indirectly? eager by default
```

すべてのノードを選択する `*` 演算子は次のように記述できます:

```yml
definition:
  method: fqn
  value: "*"
```

#### Exclude

`exclude` キーワードは、完全修飾辞書でのみサポートされます。
各辞書への引数として、または `union` の要素として渡すことができます。以下の式はどちらも同等です。

```yml
- method: tag
  value: nightly
  exclude:
    - "@tag:daily"
```

```yml
- union:
    - method: tag
      value: nightly
    - exclude:
       - method: tag
         value: daily
```

注: YAMLセレクターの`exclude`引数は、CLI引数`--exclude`とは微妙に異なります。`exclude`は常に[差集合](https://en.wikipedia.org/wiki/Complement_(set_theory))を返し、そのスコープ内では常に最後に適用されます。

複数の「yeslist」(`--select`)が渡された場合、それらは[積集合](/reference/node-selection/set-operators#intersections)ではなく[和集合](/reference/node-selection/set-operators#unions)として扱われます。複数の「nolist」(`--exclude`)が渡された場合も同様です。


#### 間接選択

原則として、dbt は、直接選択しているリソースのいずれかに関係するすべてのテストを間接的に選択します。これを「eager」間接選択と呼びます。間接選択モードを「cautious」、「buildable」、「empty」のいずれかに切り替えるには、特定の基準に対して `indirect_selection` を設定します:

```yml
- union:
    - method: fqn
      value: model_a
      indirect_selection: eager  # default: will include all tests that touch model_a
    - method: fqn
      value: model_b
      indirect_selection: cautious  # will not include tests touching model_b
                        # if they have other unselected parents
    - method: fqn
      value: model_c
      indirect_selection: buildable  # will not include tests touching model_c
                        # if they have other unselected parents (unless they have an ancestor that is selected)
    - method: fqn
      value: model_d
      indirect_selection: empty  # will include tests for only the selected node and ignore all tests attached to model_d
```

YAMLセレクターの`indirect_selection`値が指定されている場合、CLIフラグ`--indirect-selection`よりも優先されます。`indirect_selection`は選択基準ごとに個別に定義されるため、同じ定義内でeager/cautious/buildable/emptyモードを混在させて、必要な動作を正確に実現できます。`dbt ls --selector`を使用して、いつでも基準をテストできます。

間接選択の詳細については、[テスト選択の例](/reference/node-selection/test-selection-examples)を参照してください。

## 例

以下の2つの表現方法があります:


  ```bash
  $ dbt run --select @source:snowplow,tag:nightly models/export --exclude package:snowplow,config.materialized:incremental export_performance_timing
  ```

<Tabs
  defaultValue="cli_style"
  values={[
    { label: 'CLI-style', value: 'cli_style', },
    { label: 'Full YML', value: 'all_yml', },
  ]
}>

<TabItem value="cli_style">
<File name='selectors.yml'>

```yml

selectors:
  - name: nightly_diet_snowplow
    description: "Non-incremental Snowplow models that power nightly exports"
    definition:

      # Optional `union` and `intersection` keywords map to the ` ` and `,` set operators:
      union:
        - intersection:
            - '@source:snowplow'
            - 'tag:nightly'
        - 'models/export'
        - exclude:
            - intersection:
                - 'package:snowplow'
                - 'config.materialized:incremental'
            - export_performance_timing
```
</File>
</TabItem>

<TabItem value="all_yml">
<File name='selectors.yml'>

```yml
selectors:
  - name: nightly_diet_snowplow
    description: "Non-incremental Snowplow models that power nightly exports"
    definition:
      # Optional `union` and `intersection` keywords map to the ` ` and `,` set operators:
      union:
        - intersection:
            - method: source
              value: snowplow
              childrens_parents: true
            - method: tag
              value: nightly
        - method: path
          value: models/export
        - exclude:
            - intersection:
                - method: package
                  value: snowplow
                - method: config.materialized
                  value: incremental
            - method: fqn
              value: export_performance_timing
```
</File>
</TabItem>

</Tabs>

次に、ジョブ定義で次の操作を行います:

```bash
dbt run --selector nightly_diet_snowplow
```

## Default

セレクタはブール型の「default」プロパティを定義できます。セレクタに「default: true」が指定されている場合、タスクが独自の選択基準を定義していない場合、dbt はこのセレクタの基準を使用します。

ルートプロジェクトで定義されたリソースのみを選択するデフォルトセレクタを定義するとします:

```yml
selectors:
  - name: root_project_only
    description: >
        Only resources from the root project.
        Excludes resources defined in installed packages.
    default: true
    definition:
      method: package
      value: <my_root_project_name>
```

「非修飾」コマンドを実行すると、dbt は `root_project_only` で定義された選択基準を使用します。つまり、dbt はルート プロジェクトで定義されたリソースに対してのみ、ビルド/最新性チェック/コンパイル済み SQL を生成します。

```
dbt build
dbt source freshness
dbt docs generate
```

`--select`、`--exclude`、または`--selector`を使用して独自の選択基準を定義するコマンドを実行した場合、dbtはデフォルトのセレクタを無視し、代わりにフラグ基準を使用します。2つのセレクタを組み合わせることはありません。

```bash
dbt run --select  "model_a"
dbt run --exclude model_a
```

呼び出しごとに `default: true` を設定できるセレクタは1つだけです。それ以外の場合、dbt はエラーを返します。ただし、環境に応じて `default` の値を調整するには、Jinja 式を使用できます。

```yml
selectors:
  - name: default_for_dev
    default: "{{ target.name == 'dev' | as_bool }}"
    definition: ...
  - name: default_for_prod
    default: "{{ target.name == 'prod' | as_bool }}"
    definition: ...
```

### セレクタの継承

セレクタは、`selector` メソッドを介して他のセレクタの定義を再利用および拡張できます:

```yml
selectors:
  - name: foo_and_bar
    definition:
      intersection:
        - tag: foo
        - tag: bar

  - name: foo_bar_less_buzz
    definition:
      intersection:
        # reuse the definition from above
        - method: selector
          value: foo_and_bar
        # with a modification!
        - exclude:
            - method: tag
              value: buzz
```

**注:** セレクタ継承では、別のセレクタのロジックを_再利用_できますが、`parents`、`children`、`indirect_selection`などによってそのセレクタのロジックを_変更_することはできません。

`selector`メソッドは、指定されたセレクタによって返されるノードの完全なセットを返します。

## `--select` と `--selector` の違い

dbt では、[`select`](/reference/node-selection/syntax#how-does-selection-work) と `selector` は、特定のモデル、テスト、またはリソースを選択するために使用される関連概念です。以下の表は、それらの違いと、最適な使用例を示しています。

| Feature	| `--select` | `--selector` |
| ------- | ---------- | ------------- |
| Definition |	アドホック。コマンドで直接指定します。| `selectors.yml` ファイルに事前定義されています。 |
| Usage | 1 回限りまたはタスク固有のフィルタリング。| 複数回の実行に再利用可能。|
| 複雑さ | 選択基準を手動で入力する必要があります。| 複雑なロジックをカプセル化して再利用できます。|
| Flexibility	| 柔軟性は低いですが、再利用性は低くなります。 | 柔軟性は高いですが、再利用可能で構造化されたロジックに重点を置いています。|
| Example	| `dbt run --select my_model+`<br /> (`my_model` と、`+` 演算子を含むすべての下流依存関係を実行します)。 | `dbt run --selector nightly_diet_snowplow`<br /> (`selectors.yml` の `nightly_diet_snowplow` セレクターで定義されたモデルを実行します)。 |

注:
-- `--select` と `--exclude` を組み合わせて、ノードをアドホックに選択できます。
-- `--select` と `--selector` の構文はどちらも、ノード選択において基本的に同じ機能を提供します。`--select` で [グラフ演算子](/reference/node-selection/graph-operators) (`+`、`@` など) と [集合演算子](/reference/node-selection/set-operators) (`union`、`intersection` など) を使用することは、`--selector` で YAML ベースの設定を使用するのと同じです。

その他の例については、[こちらの GitHub Gist](https://gist.github.com/jeremyyeo/1aeca767e2a4f157b07955d58f8078f7) をご覧ください。
