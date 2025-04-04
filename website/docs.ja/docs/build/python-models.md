---
title: "Python モデル"
description: "Configure Python models to enhance your dbt project."
id: "python-models"
---

dbt-py モデルをサポートしているのは、[特定のデータ プラットフォーム](#specific-data-platforms) のみであることに注意してください。

次のことをお勧めします:
- この機能を提案した [元のディスカッション](https://github.com/dbt-labs/dbt-core/discussions/5261) をお読みください。
- [dbt で Python モデルを開発するためのベスト プラクティス](https://discourse.getdbt.com/t/dbt-python-model-dbt-py-best-practices/5204) に貢献してください。
- [Python モデルの次のステップ](https://github.com/dbt-labs/dbt-core/discussions/5742) に関するご意見やアイデアを共有してください。
- [dbt コミュニティ Slack](https://www.getdbt.com/community/join-the-community/) の **#dbt-core-python-models** チャネルに参加してください。


## 概要

dbt Python (`dbt-py`) モデルは、SQL では解決できないユースケースを解決するのに役立ちます。データ サイエンスと統計の最先端のパッケージを含む、オープン ソース Python エコシステムで利用可能なツールを使用して分析を実行できます。以前は、本番環境で Python 変換を実行するには、別のインフラストラクチャとオーケストレーションが必要でした。dbt で定義された Python 変換は、テスト、ドキュメント、系統に関するすべての同じ機能を備えたプロジェクト内のモデルです。


<File name='models/my_python_model.py'>

```python
import ...

def model(dbt, session):

    my_sql_model_df = dbt.ref("my_sql_model")

    final_df = ...  # stuff you can't write in SQL!

    return final_df
```

</File>

<File name='models/config.yml'>

```yml
version: 2

models:
  - name: my_python_model

    # Document within the same codebase
    description: My transformation written in Python

    # Configure in ways that feel intuitive and familiar
    config:
      materialized: table
      tags: ['python']

    # Test the results of my Python transformation
    columns:
      - name: id
        # Standard validation for 'grain' of Python results
        tests:
          - unique
          - not_null
    tests:
      # Write your own validation logic (in SQL) for Python results
      - [custom_generic_test](/best-practices/writing-custom-generic-tests)
```

</File>

<!--- TODO: how to make this image preview bigger? --->
<Lightbox src="/img/docs/building-a-dbt-project/building-models/python-models/python-model-dag.png" title="SQL + Python, together at last" style="width:200%"/>

dbt Python モデルの前提条件には、フル機能の Python ランタイムをサポートするデータ プラットフォームのアダプターの使用が含まれます。dbt Python モデルでは、すべての Python コードはプラットフォーム上でリモートで実行されます。dbt によってローカルで実行されるものはありません。私たちは、_モデル定義_ と _モデル実行_ を明確に分離することを信条としています。この方法や他の多くの方法から、dbt の Python モデルへのアプローチは、SQL でデータをモデル化する長年のアプローチを反映していることがわかります。

このガイドは、読者が dbt にある程度精通していることを前提に作成されています。dbt モデルを作成したことがない場合は、まず [dbt モデル](/docs/build/models) を読むことをお勧めします。全体を通して、Python モデルと SQL モデルの関係を示し、その違いを明確にします。

### Python モデルとは何ですか?

dbt Python モデルは、dbt ソースまたは他のモデルを読み取り、一連の変換を適用し、変換されたデータセットを返す関数です。<Term id="dataframe">DataFrame</Term> 操作は、開始点、終了状態、および途中の各ステップを定義します。

これは、dbt SQL モデルにおける <Term id="cte">CTE</Term> の役割に似ています。CTE を使用して上流データセットをプルし、一連の意味のある変換を定義 (および命名) し、最終的な `select` ステートメントで終了します。コンパイルされたバージョンの dbt SQL モデルを実行して、結果のビューまたはテーブルに含まれるデータを確認できます。`dbt run` を実行すると、dbt はそのクエリを `create view`、`create table`、またはより複雑な DDL でラップして、その結果をデータベースに保存します。

最終的な `select` ステートメントの代わりに、各 Python モデルは最終的な DataFrame を返します。各 DataFrame 操作は「遅延評価」されます。開発中は、`.show()` や `.head()` などのメソッドを使用して、そのデータをプレビューできます。Python モデルを実行すると、最終的な DataFrame の完全な結果がデータ ウェアハウスにテーブルとして保存されます。

dbt Python モデルは、SQL モデルとほぼ同じ構成オプションにアクセスできます。モデルをテストして文書化したり、`tags` や `meta` プロパティを追加したり、他のユーザーに結果へのアクセスを許可したりできます。モデルは、名前、ファイル パス、構成、別のモデルの上流または下流にあるかどうか、以前のプロジェクト状態と比較して変更されているかどうかで選択できます。

### Python モデルの定義

各 Python モデルは、`models/` フォルダーの `.py` ファイルにあります。このファイルは、2 つのパラメーターを受け取る **`model()`** という関数を定義します。
- **`dbt`**: dbt Core によってコンパイルされ、各モデルに固有のクラス。これにより、dbt プロジェクトと DAG のコンテキストで Python コードを実行できます。
- **`session`**: データ プラットフォームの Python バックエンドへの接続を表すクラス。セッションは、テーブルを DataFrame として読み込んだり、DataFrame をテーブルに書き戻したりするために必要です。PySpark では、慣例により、`SparkSession` は `spark` という名前で、グローバルに使用できます。プラットフォーム間の一貫性を保つために、常に `session` という明示的な引数として `model` 関数に渡します。

`model()` 関数は、単一の DataFrame を返す必要があります。Snowpark (Snowflake) では、これは Snowpark または pandas DataFrame になります。 PySpark (Databricks + BigQuery) 経由では、Spark、pandas、または pandas-on-Spark DataFrame になります。pandas とネイティブ DataFrame の選択の詳細については、[DataFrame API + 構文](#dataframe-api-and-syntax) を参照してください。

`dbt run --select python_model` を実行すると、dbt は両方の引数 (`dbt` と `session`) を準備して渡します。必要なのは関数を定義することだけです。すべての Python モデルは次のようになります。

<File name='models/my_python_model.py'>

```python
def model(dbt, session):

    ...

    return final_df
```

</File>


### 他のモデルを参照する

Python モデルは、dbt の変換の有向非巡回グラフ (DAG) に完全に参加します。Python モデル内で `dbt.ref()` メソッドを使用して、他のモデル (SQL または Python) からデータを読み取ります。生のソース テーブルから直接読み取る場合は、`dbt.source()` を使用します。これらのメソッドは、上流のソース、モデル、シード、またはスナップショットを指す DataFrames を返します。

<File name='models/my_python_model.py'>

```python
def model(dbt, session):

    # DataFrame representing an upstream model
    upstream_model = dbt.ref("upstream_model_name")

    # DataFrame representing an upstream source
    upstream_source = dbt.source("upstream_source_name", "table_name")

    ...
```

</File>

もちろん、ダウンストリーム SQL モデルで Python モデルを `ref()` することもできます。

<File name='models/downstream_model.sql'>

```sql
with upstream_python_model as (

    select * from {{ ref('my_python_model') }}

),

...
```

</File>

:::caution

[ephemeral](/docs/build/materializations#ephemeral) モデルの参照は現在サポートされていません ([機能リクエスト](https://github.com/dbt-labs/dbt-core/issues/7288) を参照)
:::

<VersionBlock firstVersion="1.8">

dbt バージョン 1.8 以降、Python モデルは Python f 文字列内での動的構成もサポートします。これにより、Python コード内で直接、より繊細で動的なモデル構成が可能になります。例:

<File name='models/my_python_model.py'>

```python
# Previously, attempting to access a configuration value like this would result in None
print(f"{dbt.config.get('my_var')}")  # Output before change: None

# Now you can access the actual configuration value
# Assuming 'my_var' is configured to 5 for the current model
print(f"{dbt.config.get('my_var')}")  # Output after change: 5
```

これは、Python モデル内で `dbt.config.get()` を使用して、構成値が Python f 文字列内で効果的に取得および使用可能であることを保証できることも意味します。

</File>
</VersionBlock>

## Python モデルの設定

SQL モデルと同様に、Python モデルを構成する方法は 3 つあります:
1. `dbt_project.yml` で、一度に複数のモデルを構成できます
2. `models/` ディレクトリ内の専用の `.yml` ファイルで
3. モデルの `.py` ファイル内で、`dbt.config()` メソッドを使用します

`dbt.config()` メソッドを呼び出すと、`.sql` モデル ファイルの `{{ config() }}` マクロと同様に、`.py` ファイル内でモデルの構成が設定されます:

<File name='models/my_python_model.py'>

```python
def model(dbt, session):

    # setting configuration
    dbt.config(materialized="table")
```

</File>

`dbt.config()` メソッドで設定できる複雑さには限界があります。このメソッドは、リテラル値 (文字列、ブール値、数値型) と動的設定のみを受け入れます。別の関数やより複雑なデータ構造を渡すことはできません。これは、dbt が Python コードを実行せずにモデルを解析しながら `config()` への引数を静的に分析するためです。より複雑な設定を設定する必要がある場合は、YAML ファイルの [`config` プロパティ](/reference/resource-properties/config) を使用して定義することをお勧めします。

#### プロジェクトコンテキストへのアクセス

dbt Python モデルは、コンパイルされたコードをレンダリングするために Jinja を使用しません。Python モデルは、SQL モデルと比較して、グローバル プロジェクト コンテキストへのアクセスが制限されています。そのコンテキストは、`model()` 関数に引数として渡される `dbt` クラスから利用可能になります。

`dbt` クラスは、すぐに使用できる次の機能をサポートします:
- 他のリソースの場所を参照する DataFrame を返す: `dbt.ref()` + `dbt.source()`
- 現在のモデルのデータベースの場所にアクセスする: `dbt.this()` (`dbt.this.database`、`.schema`、`.identifier` も)
- 現在のモデルの実行が増分であるかどうかを判断する: `dbt.is_incremental`

[モデルの構成](/reference/model-configs) で構成された後、`dbt.config.get()` を使用して「取得」することで、このコンテキストを拡張できます。 dbt v1.8 以降、`dbt.config.get()` メソッドは Python モデル内の構成への動的アクセスをサポートし、モデル ロジックの柔軟性を高めます。これには、`var`、`env_var`、`target` などの入力が含まれます。モデルの条件付きロジックにこれらの値を使用する場合は、専用の YAML ファイル構成を使用して設定する必要があります:

<File name='models/config.yml'>

```yml
version: 2

models:
  - name: my_python_model
    config:
      materialized: table
      target_name: "{{ target.name }}"
      specific_var: "{{ var('SPECIFIC_VAR') }}"
      specific_env_var: "{{ env_var('SPECIFIC_ENV_VAR') }}"
```

</File>

次に、モデルの Python コード内で、`dbt.config.get()` 関数を使用して、設定されている構成の値に _アクセス_ します:

<File name='models/my_python_model.py'>

```python
def model(dbt, session):
    target_name = dbt.config.get("target_name")
    specific_var = dbt.config.get("specific_var")
    specific_env_var = dbt.config.get("specific_env_var")

    orders_df = dbt.ref("fct_orders")

    # limit data in dev
    if target_name == "dev":
        orders_df = orders_df.limit(500)
```

</File>

<VersionBlock firstVersion="1.8">

#### 動的構成

Python モデルを構成する既存の方法に加えて、f 文字列を使用して Python モデル内の `dbt.config()` で設定された構成値に動的にアクセスすることもできます。これにより、カスタム ロジックと構成管理の可能性が広がります。

<File name='models/my_python_model.py'>

```python
def model(dbt, session):
    dbt.config(materialized="table")
    
    # Dynamic configuration access within Python f-strings, 
    # which allows for real-time retrieval and use of configuration values.
    # Assuming 'my_var' is set to 5, this will print: Dynamic config value: 5
    print(f"Dynamic config value: {dbt.config.get('my_var')}")
```

</File>
</VersionBlock>

### マテリアライゼーション

Python モデルでは、次のマテリアライゼーションがサポートされています:
- `table` (デフォルト)
- `incremental`

増分 Python モデルでは、SQL のモデルと同じ [増分戦略](/docs/build/incremental-strategy) がすべてサポートされています。サポートされる具体的な戦略は、アダプタによって異なります。たとえば、増分モデルは、Dataproc を使用した BigQuery で `merge` 増分戦略でサポートされていますが、`insert_overwrite` 戦略はまだサポートされていません。

Python モデルは、`view` または `ephemeral` としてマテリアライゼーションできません。Python は、モデル以外のリソースタイプ (テストやスナップショットなど) ではサポートされていません。

SQL モデルなどの増分モデルでは、受信テーブルを新しいデータ行のみにフィルタリングする必要があります。

<Tabs>

<TabItem value="Snowpark">

<File name='models/my_python_model.py'>

```python
import snowflake.snowpark.functions as F

def model(dbt, session):
    dbt.config(materialized = "incremental")
    df = dbt.ref("upstream_table")

    if dbt.is_incremental:

        # only new rows compared to max in current table
        max_from_this = f"select max(updated_at) from {dbt.this}"
        df = df.filter(df.updated_at >= session.sql(max_from_this).collect()[0][0])

        # or only rows from the past 3 days
        df = df.filter(df.updated_at >= F.dateadd("day", F.lit(-3), F.current_timestamp()))

    ...

    return df
```

</File>

</TabItem>

<TabItem value="PySpark">

<File name='models/my_python_model.py'>

```python
import pyspark.sql.functions as F

def model(dbt, session):
    dbt.config(materialized = "incremental")
    df = dbt.ref("upstream_table")

    if dbt.is_incremental:

        # only new rows compared to max in current table
        max_from_this = f"select max(updated_at) from {dbt.this}"
        df = df.filter(df.updated_at >= session.sql(max_from_this).collect()[0][0])

        # or only rows from the past 3 days
        df = df.filter(df.updated_at >= F.date_add(F.current_timestamp(), F.lit(-3)))

    ...

    return df
```

</File>

</TabItem>

</Tabs>

## Python固有の機能

### 関数の定義

`model` 関数を定義することに加えて、Python モデルは他の関数をインポートしたり、独自の関数を定義したりすることができます。以下は、Snowpark でカスタム `add_one` 関数を定義する例です:

<File name='models/my_python_model.py'>

```python
def add_one(x):
    return x + 1

def model(dbt, session):
    dbt.config(materialized="table")
    temps_df = dbt.ref("temperatures")

    # warm things up just a little
    df = temps_df.withColumn("degree_plus_one", add_one(temps_df["degree"]))
    return df
```

</File>

現在、1 つの dbt モデルで定義された Python 関数を他のモデルにインポートして再利用することはできません。検討されている潜在的なパターンについては、[コードの再利用](#code-reuse) を参照してください。

### PyPIパッケージの使用

また、サードパーティのパッケージがデータ プラットフォーム上の Python ランタイムにインストールされ、使用可能であれば、サードパーティのパッケージに依存する関数を定義することもできます。[特定のデータ プラットフォーム](#specific-data-platforms) の「パッケージのインストール」に関する注記を参照してください。

この例では、`holidays` パッケージを使用して、特定の日付がフランスの休日かどうかを判断します。以下のコードでは、プラットフォーム間での簡潔性と一貫性を保つために pandas API を使用しています。正確な構文、およびマルチノード処理のためのリファクタリングの必要性は、依然として異なります。
<Tabs>

<TabItem value="Snowpark">

<File name='models/my_python_model.py'>

```python
import holidays

def is_holiday(date_col):
    # Chez Jaffle
    french_holidays = holidays.France()
    is_holiday = (date_col in french_holidays)
    return is_holiday

def model(dbt, session):
    dbt.config(
        materialized = "table",
        packages = ["holidays"]
    )

    orders_df = dbt.ref("stg_orders")

    df = orders_df.to_pandas()

    # apply our function
    # (columns need to be in uppercase on Snowpark)
    df["IS_HOLIDAY"] = df["ORDER_DATE"].apply(is_holiday)
    df["ORDER_DATE"].dt.tz_localize('UTC') # convert from Number/Long to tz-aware Datetime

    # return final dataset (Pandas DataFrame)
    return df
```

</File>

</TabItem>

<TabItem value="PySpark">

<File name='models/my_python_model.py'>

```python
import holidays

def is_holiday(date_col):
    # Chez Jaffle
    french_holidays = holidays.France()
    is_holiday = (date_col in french_holidays)
    return is_holiday

def model(dbt, session):
    dbt.config(
        materialized = "table",
        packages = ["holidays"]
    )

    orders_df = dbt.ref("stg_orders")

    df = orders_df.to_pandas_on_spark()  # Spark 3.2+
    # df = orders_df.toPandas() in earlier versions

    # apply our function
    df["is_holiday"] = df["order_date"].apply(is_holiday)

    # convert back to PySpark
    df = df.to_spark()               # Spark 3.2+
    # df = session.createDataFrame(df) in earlier versions

    # return final dataset (PySpark DataFrame)
    return df
```

</File>

</TabItem>

</Tabs>

#### パッケージの設定

必要なパッケージとバージョンを構成して、dbt がプロジェクト メタデータでそれらを追跡できるようにすることをお勧めします。この構成は、一部のプラットフォームでの実装に必要です。特定のバージョンのパッケージが必要な場合は、それを指定します。

<File name='models/my_python_model.py'>

```python
def model(dbt, session):
    dbt.config(
        packages = ["numpy==1.23.1", "scikit-learn"]
    )
```

</File>

<File name='models/config.yml'>

```yml
version: 2

models:
  - name: my_python_model
    config:
      packages:
        - "numpy==1.23.1"
        - scikit-learn
```

</File>

#### ユーザー定義関数 (UDF)

`@udf` デコレータまたは `udf` 関数を使用して「匿名」関数を定義し、`model` 関数の DataFrame 変換内でそれを呼び出すことができます。これは、特にそれらの関数がサードパーティ パッケージからの入力を必要とする場合に、より複雑な関数を DataFrame 操作として適用するための一般的なパターンです。
- [Snowpark Python: UDF の作成](https://docs.snowflake.com/en/developer-guide/snowpark/python/creating-udfs.html)
- [PySpark 関数: udf](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.functions.udf.html)

<Tabs>

<TabItem value="Snowpark">

<File name='models/my_python_model.py'>

```python
import snowflake.snowpark.types as T
import snowflake.snowpark.functions as F
import numpy

def register_udf_add_random():
    add_random = F.udf(
        # use 'lambda' syntax, for simple functional behavior
        lambda x: x + numpy.random.normal(),
        return_type=T.FloatType(),
        input_types=[T.FloatType()]
    )
    return add_random

def model(dbt, session):

    dbt.config(
        materialized = "table",
        packages = ["numpy"]
    )

    temps_df = dbt.ref("temperatures")

    add_random = register_udf_add_random()

    # warm things up, who knows by how much
    df = temps_df.withColumn("degree_plus_random", add_random("degree"))
    return df
```

</File>

**注:** Snowpark の制限により、現在、ストアド プロシージャ内、つまり dbt Python モデル内に複雑な名前の UDF を登録することはできません。今後のリリースでは、プロジェクト/DAG リソース タイプとして Python UDF のネイティブ サポートを追加する予定です。当面、バッチ API 経由で「ベクトル化された」Python UDF を作成する場合は、次のいずれかをお勧めします。
- SQL マクロ内に [`create function`](https://docs.snowflake.com/en/developer-guide/udf/python/udf-python-batch.html) を記述して、フックまたは実行操作として実行する
- Python モデル コード内で [ステージングされたファイルから登録](https://docs.snowflake.com/en/developer-guide/snowpark/python/creating-udfs#creating-a-udf-from-a-python-source-file) する

</TabItem>

<TabItem value="PySpark">

<File name='models/my_python_model.py'>

```python
import pyspark.sql.types as T
import pyspark.sql.functions as F
import numpy

# use a 'decorator' for more readable code
@F.udf(returnType=T.DoubleType())
def add_random(x):
    random_number = numpy.random.normal()
    return x + random_number

def model(dbt, session):
    dbt.config(
        materialized = "table",
        packages = ["numpy"]
    )

    temps_df = dbt.ref("temperatures")

    # warm things up, who knows by how much
    df = temps_df.withColumn("degree_plus_random", add_random("degree"))
    return df
```

</File>

</TabItem>

</Tabs>

#### コードの再利用

現在、1 つの dbt モデルで定義された Python 関数を他のモデルにインポートして再利用することはできません。これは dbt Labs がサポートしたいと考えていることであり、検討しているパターンは 2 つあります。

- **「名前付き」UDF** の作成と登録 - このプロセスはデータ プラットフォームごとに異なり、パフォーマンス上の制限もあります。たとえば、Snowpark は、並列実行できる pandas のような関数の [ベクトル化された UDF](https://docs.snowflake.com/en/developer-guide/udf/python/udf-python-batch.html) をサポートしています。
- **プライベート Python パッケージ** - パブリック PyPI パッケージから再利用可能な関数をインポートすることに加えて、多くのデータ プラットフォームでは、カスタム Python アセットをアップロードしてパッケージとして登録することがサポートされています。アップロード プロセスはプラットフォームごとに異なりますが、コードの実際の `import` は同じです。

:::note ❓ dbt の質問

- dbt は UDF を抽象化する役割を持つべきでしょうか? dbt は新しいタイプの DAG ノードである `function` をサポートすべきでしょうか? 主な使用例は、Python モデル間でのコードの再利用でしょうか、それとも SQL モデルから呼び出せる Python 言語関数の定義でしょうか?
- プライベート Python アセットをアップロードまたは初期化するときに、dbt はユーザーをどのようにサポートできますか? これは `dbt deps` の新しい形式でしょうか?
- カスタム関数をテストしたいユーザーを dbt はどのようにサポートできますか? UDF として定義されている場合: データベースで「ユニット テスト」? パッケージ内の「純粋な」関数の場合: `pytest` の採用を推奨?

💬 ディスカッション: ["Python モデル: dbt でのパッケージ、アーティファクト/オブジェクト ストレージ、および UDF 管理"](https://github.com/dbt-labs/dbt-core/discussions/5741)
:::

### DataFrame APIと構文

過去 10 年間、Python で [データ変換](https://www.getdbt.com/analytics-engineering/transformation/) を記述するほとんどの人は、共通の抽象化として <Term id="dataframe">DataFrame</Term> を採用してきました。dbt はこの規則に従い、`ref()` と `source()` を DataFrame として返します。また、すべての Python モデルが DataFrame を返すことを期待しています。

DataFrame は 2 次元データ構造 (行と列) です。そのデータを変換したり、既存の列に対して実行された計算から新しい列を作成したりするための便利なメソッドをサポートしています。また、ローカルまたはノートブックで開発中にデータをプレビューするための便利な方法も提供します。

合意はそこで終わります。DataFrame には独自の構文と API を持つフレームワークが多数あります。[pandas](https://pandas.pydata.org/docs/) ライブラリはオリジナルの DataFrame API の 1 つを提供し、その構文は新しいデータ プロフェッショナルが習得する最も一般的なものです。新しい DataFrame API のほとんどは pandas スタイルの構文と互換性がありますが、完全な相互運用性を提供できるものはほとんどありません。これは、独自の DataFrame API を持つ Snowpark と PySpark にも当てはまります。

Python モデルを開発するときに、次のような疑問が湧いてくるでしょう。

**なぜ pandas か?** &mdash; これは DataFrames の最も一般的な API です。これにより、サンプリングされたデータを簡単に探索し、ローカルで変換を開発できます。コードをそのまま dbt モデルに「昇格」し、小規模なデータセットの運用環境で実行できます。

**なぜ pandas ではないか?** &mdash; パフォーマンス。pandas は「単一ノード」変換を実行しますが、これは最新のデータ ウェアハウスが提供する並列処理と分散コンピューティングのメリットを享受できません。これは、大規模なデータセットを操作するときにすぐに問題になります。一部のデータ プラットフォームでは、pandas DataFrame API を使用して記述されたコードの最適化がサポートされているため、大幅なリファクタリングは必要ありません。たとえば、[PySpark 上の pandas](https://spark.apache.org/docs/latest/api/python/getting_started/quickstart_ps.html) は、並列処理を活用しながら同じ API を使用して、pandas 機能の 95% をサポートします。

:::note ❓ dbt の質問
- 新しい dbt Python モデルを開発する場合、迅速な反復とリファクタリングのために pandas スタイルの構文を推奨すべきでしょうか?
- さまざまなデータ エンジンとベンダー固有の API にわたって魅力的な抽象化を提供するオープン ソース ライブラリはどれですか?
- dbt は、それら全体にわたる標準化において長期的な役割を果たすことを試みるべきでしょうか?

💬 ディスカッション: ["Python モデル: pandas の問題 (および可能な解決策)"](https://github.com/dbt-labs/dbt-core/discussions/5738)
:::

## 制限事項

Python モデルには、SQL モデルにはない機能があります。また、SQL モデルと比較していくつかの欠点もあります。

- **時間とコスト。** Python モデルは SQL モデルよりも実行速度が遅く、それらを実行するクラウド リソースは高価になる場合があります。Python を実行するには、より汎用的なコンピューティングが必要です。そのコンピューティングは、SQL モデルとは別のサービスまたはアーキテクチャに存在する場合があります。**ただし、** 統一された系統、テスト、およびドキュメントを備えた dbt を介して Python モデルを展開することは、人間の観点から、**劇的に**高速で安価であると考えています。比較すると、本番環境で Python 変換を調整するために別のインフラストラクチャを立ち上げ、dbt と統合するためのさまざまなツールを構築すると、はるかに時間がかかり、コストがかかります。
- **構文の違い** はさらに顕著です。長年にわたり、dbt はディスパッチ パターンや `dbt_utils` などのパッケージを介して、一般的なデータ ウェアハウス間の SQL 方言の違いを抽象化するために多くのことを行ってきました。Python は **はるかに** 広い分野を提供します。 SQL で何かを実行する方法が 5 つある場合、Python でそれを記述する方法は 500 通りあり、パフォーマンスや標準への準拠はそれぞれ異なります。これらのオプションは圧倒的です。dbt のメンテナーとして、私たちはこの問題に取り組む最先端のプロジェクトから学び、開発しながらガイダンスを共有していきます。
- **これらの機能は非常に新しいものです。** データ ウェアハウスが新しい機能を開発するにつれて、Python 変換を展開するためのより安価で高速で直感的なメカニズムが提供されると予想されます。**将来のリリースで Python モデルを実行するための基盤となる実装を変更する権利を留保します。** お客様に対する私たちのコミットメントは、ここで提供しているドキュメント化された機能とガイダンスに従って、モデルの `.py` ファイル内のコードに関するものです。
- **`print()` サポートがありません。** データ プラットフォームは、dbt の監視なしに Python モデルを実行してコンパイルします。つまり、Python の組み込み [`print()`](https://docs.python.org/3/library/functions.html#print) 関数などのコマンドの出力は dbt のログに表示されません。

- <Expandable alt_header="Python モデルで print() を使用する代わりに">

    以下では、データフレーム列へのメッセージの書き込みなど、デバッグに使用できるその他の方法について説明します。

    - プラットフォーム ログの使用: データ プラットフォームのログを使用して、Python モデルをデバッグします。

    - ログをデータフレームとして返す: ログを含むデータフレームを作成し、ウェアハウスに組み込みます。

    - DuckDB を使用してローカルで開発する: デプロイする前に、DuckDB を使用してローカルでモデルをテストおよびデバッグします。

    Python モデルでのデバッグの例を次に示します。

    ```python
    def model(dbt, session):
        dbt.config(
            materialized = "table"
        )
    
        df = dbt.ref("my_source_table").df()
    
        # One option for debugging: write messages to temporary table column
        # Pros: visibility
        # Cons: won't work if table isn't building for some reason
        msg = "something"
        df["debugging"] = f"My debug message here: {msg}"
    
        return df
    ```
    </Expandable>

一般的なルールとして、SQL と Python のどちらでも同じようにうまく記述できる変換がある場合、適切に記述された SQL の方が望ましいと考えられます。SQL の方が、より多くの同僚がアクセスしやすく、大規模にパフォーマンスの高いコードを簡単に記述できます。SQL では記述できない変換がある場合、または 10 行のエレガントで適切に注釈が付けられた Python によって 1000 行の読みにくい Jinja-SQL を節約できる場合は、Python を使用することをお勧めします。

## 特定のデータプラットフォーム {#specific-data-platforms}

Python モデルは、Snowflake、Databricks、BigQuery/GCP (Dataproc 経由) など、多くのアダプタでサポートされています。Databricks と GCP の Dataproc はどちらも、処理フレームワークとして PySpark を使用します。Snowflake は独自のフレームワークである Snowpark を使用しますが、これは PySpark と多くの類似点があります。

<Tabs>

<TabItem value="Snowflake">

**追加の設定:** Anaconda パッケージを使用するには、[Snowflake サードパーティ規約を承認して同意する](https://docs.snowflake.com/en/developer-guide/udf/python/udf-python-packages.html#getting-started)必要があります。

**パッケージのインストール:** Snowpark は、Anaconda を介していくつかの一般的なパッケージをサポートしています。詳細については、[完全なリスト](https://repo.anaconda.com/pkgs/snowflake/)を参照してください。パッケージは、モデルの実行時にインストールされます。モデルによってパッケージの依存関係が異なる場合があります。サードパーティのパッケージを使用する場合、Snowflake では、同時ユーザー数が多いウェアハウスではなく、専用の仮想ウェアハウスを使用して最高のパフォーマンスを得ることをお勧めします。

**Python バージョン:** 別の Python バージョンを指定するには、次の構成を使用します。

```python
def model(dbt, session):
    dbt.config(
        materialized = "table",
        python_version="3.11"
    )
```

`python_version` 構成を使用すると、[Python バージョン](https://docs.snowflake.com/en/developer-guide/snowpark/python/setup) 3.9、3.10、または 3.11 で Snowpark モデルを実行できます。

<VersionBlock firstVersion="1.8">

**外部アクセスの統合とシークレット**: dbt Python モデル内で外部 API をクエリするには、Snowflake の [外部アクセス](https://docs.snowflake.com/en/developer-guide/external-network-access/external-network-access-overview) と [シークレット](https://docs.snowflake.com/en/developer-guide/external-network-access/secret-api-reference) を併用します。使用できる追加の構成を次に示します。

```python
import pandas
import snowflake.snowpark as snowpark

def model(dbt, session: snowpark.Session):
    dbt.config(
        materialized="table",
        secrets={"secret_variable_name": "test_secret"},
        external_access_integrations=["test_external_access_integration"],
    )
    import _snowflake
    return session.create_dataframe(
        pandas.DataFrame(
            [{"secret_value": _snowflake.get_generic_secret_string('secret_variable_name')}]
        )
    )
```

</VersionBlock>

**ドキュメント:** ["開発者ガイド: Snowpark Python"](https://docs.snowflake.com/en/developer-guide/snowpark/python/index.html)

#### サードパーティのSnowflakeパッケージ

Snowflake Anaconda で利用できないサードパーティの Snowflake パッケージを使用するには、[この例](https://docs.snowflake.com/en/developer-guide/udf/python/udf-python-packages#importing-packages-through-a-snowflake-stage) に従ってパッケージをアップロードし、dbt Python モデルの `imports` 設定を構成して、Snowflake ステージングの zip ファイルを参照するようにします。

以下は、Python モデルでの `imports` の使用を含む、zip ファイルを使用した完全な構成例です:

```python

def model(dbt, session):
    # Configure the model
    dbt.config(
        materialized="table",
        imports=["@mystage/mycustompackage.zip"],  # Specify the external package location
    )
    
    # Example data transformation using the imported package
    # (Assuming `some_external_package` has a function we can call)
    data = {
        "name": ["Alice", "Bob", "Charlie"],
        "score": [85, 90, 88]
    }
    df = pd.DataFrame(data)

    # Process data with the external package
    df["adjusted_score"] = df["score"].apply(lambda x: some_external_package.adjust_score(x))
    
    # Return the DataFrame as the model output
    return df

```

この構成の使用に関する詳細については、Snowflake の Anaconda チャネルで公開されていない他の Python パッケージを Snowpark にアップロードして使用する方法については、[Snowflake のドキュメント](https://community.snowflake.com/s/article/how-to-use-other-python-packages-in-snowpark)を参照してください。


</TabItem>

<TabItem value="Databricks">

**送信方法:** Databricks は、それぞれ相対的な利点を持つ、PySpark コードを送信するためのいくつかの異なるメカニズムをサポートしています。反復的な開発をサポートするのに適したものもあれば、低コストの運用展開をサポートするのに適したものもあります。オプションは次のとおりです:
- `all_purpose_cluster` (デフォルト): dbt は、接続プロファイルまたはこの特定のモデルで `cluster` として構成されたクラスター ID を使用して、Python モデルを実行します。これらのクラスターはコストがかかりますが、応答性もはるかに高くなります。開発の反復を高速化するには、対話型の汎用クラスターを使用することをお勧めします。
  - `create_notebook: True`: dbt は、モデルのコンパイル済み PySpark コードを名前空間 `/Shared/dbt_python_model/{schema}` のノートブックにアップロードします。ここで、`{schema}` はモデル用に構成されたスキーマです。そして、そのノートブックを実行して、多目的クラスターを使用して実行します。この方法の利点は、モデルを実行した直後に、Databricks UI でノートブックを簡単に開いてデバッグや微調整を行えることです。再実行する前に、変更内容を dbt `.py` モデル コードにコピーすることを忘れないでください。
  - `create_notebook: False` (既定値): dbt は [コマンド API](https://docs.databricks.com/dev-tools/api/1.2/index.html#run-a-command) を使用します。これは若干高速です。
- `job_cluster`: dbt は、モデルのコンパイル済み PySpark コードを名前空間 `/Shared/dbt_python_model/{schema}` のノートブックにアップロードします。ここで、`{schema}` はモデル用に構成されたスキーマです。そして、そのノートブックを実行して、短期間のジョブ クラスターを使用して実行します。Python モデルごとに、Databricks はクラスターを起動し、モデルの PySpark 変換を実行してから、クラスターを停止する必要があります。そのため、ジョブ クラスターはモデル実行の前後に時間がかかりますが、コストも低いため、運用環境で長時間実行される Python モデルにはこれをお勧めします。 `job_cluster` 送信方法を使用するには、[JobRunsSubmit API](https://docs.databricks.com/dev-tools/api/latest/jobs.html#operation/JobsRunsSubmit) で定義されているように、`new_cluster` のキー値プロパティを定義する `job_cluster_config` を使用してモデルを構成する必要があります。

各モデルの「送信方法」は、設定を提供するすべての標準的な方法で設定できます:

```python
def model(dbt, session):
    dbt.config(
        submission_method="all_purpose_cluster",
        create_notebook=True,
        cluster_id="abcd-1234-wxyz"
    )
    ...
```
```yml
version: 2
models:
  - name: my_python_model
    config:
      submission_method: job_cluster
      job_cluster_config:
        spark_version: ...
        node_type_id: ...
```
```yml
# dbt_project.yml
models:
  project_name:
    subfolder:
      # set defaults for all .py models defined in this subfolder
      +submission_method: all_purpose_cluster
      +create_notebook: False
      +cluster_id: abcd-1234-wxyz
```

構成されていない場合、`dbt-spark` は組み込みのデフォルト、つまりノートブックを作成せずに (接続プロファイルの `cluster` に基づく) 汎用クラスターを使用します。`dbt-databricks` アダプターは、`http_path` で構成されたクラスターをデフォルトとして使用します。Databricks プロジェクトでは、Python モデルのクラスターを明示的に構成することをお勧めします。

**パッケージのインストール:** 汎用クラスターを使用する場合は、Python モデルの実行に使用するパッケージをインストールすることをお勧めします。

**ドキュメント:**
- [PySpark DataFrame 構文](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.html)
- [Databricks: DataFrame の概要 - Python](https://docs.databricks.com/spark/latest/dataframes-datasets/introduction-to-dataframes-python.html)

</TabItem>

<TabItem value="BigQuery">

`dbt-bigquery` アダプタは、Dataproc というサービスを使用して、Python モデルを PySpark ジョブとして送信します。その Python/PySpark コードは、BigQuery のテーブルとビューから読み取り、Dataproc ですべての計算を実行し、最終結果を BigQuery に書き戻します。

**送信方法。** Dataproc は、`serverless` と `cluster` の 2 つの送信方法をサポートしています。Dataproc Serverless では、準備済みのクラスタが不要なため、手間とコストを節約できますが、起動に時間がかかり、使用可能な構成の点ではるかに制限されています。たとえば、Dataproc Serverless は、`pandas`、`numpy`、`scikit-learn` は含まれていますが、Python パッケージの小さなセットのみをサポートしています。(完全なリストについては、[こちら](https://cloud.google.com/dataproc-serverless/docs/guides/custom-containers#example_custom_container_image_build) の「次のパッケージは、デフォルトのイメージにインストールされています」を参照してください)。一方、事前に Dataproc クラスタを作成しておくと、クラスタの構成を微調整し、必要な PyPI パッケージをインストールして、より高速で応答性の高いランタイムのメリットを享受できます。

自分または組織が管理する専用の Dataproc クラスタでは、`cluster` 送信方法を使用します。Spark クラスタの管理を回避するには、`serverless` 送信方法を使用します。後者の方が開始が早いかもしれませんが、どちらも本番環境で有効です。

**追加の設定:**
- [Cloud Storage バケット](https://cloud.google.com/storage/docs/creating-buckets) を作成するか、既存のものを使用します
- プロジェクト + リージョンで Dataproc API を有効にします
- `cluster` 送信方法を使用する場合: [Spark BigQuery コネクタ初期化アクション](https://github.com/GoogleCloudDataproc/initialization-actions/tree/master/connectors#bigquery-connectors) を使用して、[Dataproc クラスタ](https://cloud.google.com/dataproc/docs/guides/create-cluster) を作成するか、既存のものを使用します。(Google では、スクリーンショットに示されているサンプル バージョンを使用するのではなく、アクションを独自の Cloud Storage バケットにコピーすることを推奨しています)

<Lightbox src="/img/docs/building-a-dbt-project/building-models/python-models/dataproc-connector-initialization.png" title="Add the Spark BigQuery connector as an initialization action"/>

Dataproc で Python モデルを実行するには、次の構成が必要です。これらを [BigQuery プロファイル](/docs/core/connect-data-platform/bigquery-setup#running-python-models-on-dataproc) に追加するか、特定の Python モデルで構成できます。
- `gcs_bucket`: dbt がモデルのコンパイル済み PySpark コードをアップロードするストレージ バケット。
- `dataproc_region`: Dataproc を有効にした GCP リージョン (例: `us-central1`)。
- `dataproc_cluster_name`: Python モデルの実行 (PySpark ジョブの実行) に使用する Dataproc クラスタの名前。`submission_method: cluster` の場合にのみ必要です。

```python
def model(dbt, session):
    dbt.config(
        submission_method="cluster",
        dataproc_cluster_name="my-favorite-cluster"
    )
    ...
```
```yml
version: 2
models:
  - name: my_python_model
    config:
      submission_method: serverless
```

Dataproc Serverless で実行される Python モデルは、[BigQuery プロファイル](/docs/core/connect-data-platform/bigquery-setup#running-python-models-on-dataproc) でさらに構成できます。

dbt Python モデルを実行するすべてのユーザーまたはサービス アカウントには、必要な BigQuery 権限に加えて、次の権限が必要です ([ドキュメント](https://cloud.google.com/dataproc/docs/concepts/iam/iam)):
```
dataproc.batches.create
dataproc.clusters.use
dataproc.jobs.create
dataproc.jobs.get
dataproc.operations.get
dataproc.operations.list
storage.buckets.get
storage.objects.create
storage.objects.delete
```

**パッケージのインストール:**

Dataproc へのサードパーティ パッケージのインストールは、[クラスタ](https://cloud.google.com/dataproc/docs/guides/create-cluster) か [サーバーレス](https://cloud.google.com/dataproc-serverless/docs) かによって異なります。

- **Dataproc クラスタ** - Google では、初期化アクションを介してクラスタを作成するときに Python パッケージをインストールすることを推奨しています:
    - [初期化アクションの使用方法](https://github.com/GoogleCloudDataproc/initialization-actions/blob/master/README.md#how-initialization-actions-are-used)
    - [`pip` または `conda` 経由でインストールするためのアクション](https://github.com/GoogleCloudDataproc/initialization-actions/tree/master/python)

    [クラスタ プロパティを定義](https://cloud.google.com/dataproc/docs/tutorials/python-configuration#image_version_20): `dataproc:pip.packages` または `dataproc:conda.packages` することで、クラスタ作成時にパッケージをインストールすることもできます。

- **Dataproc Serverless** - Google では、サードパーティ パッケージをインストールするには [カスタム Docker イメージ](https://cloud.google.com/dataproc-serverless/docs/guides/custom-containers) を使用することをおすすめしています。イメージは [Google Artifact Registry](https://cloud.google.com/artifact-registry/docs) でホストする必要があります。その後、dbt プロファイルでイメージ パスを指定することで使用できます:
    
    <File name='profiles.yml'>
    ```yml
    my-profile:
        target: dev
        outputs:
            dev:
            type: bigquery
            method: oauth
            project: abc-123
            dataset: my_dataset
            
            # for dbt Python models to be run on Dataproc Serverless
            gcs_bucket: dbt-python
            dataproc_region: us-central1
            submission_method: serverless
            dataproc_batch:
                runtime_config:
                    container_image: {HOSTNAME}/{PROJECT_ID}/{IMAGE}:{TAG}
    ```


    </File>

<Lightbox src="/img/docs/building-a-dbt-project/building-models/python-models/dataproc-pip-packages.png" title="Adding packages to install via pip at cluster startup"/>

**ドキュメント:**

- [Dataproc の概要](https://cloud.google.com/dataproc/docs/concepts/overview)
- [Dataproc クラスタを作成する](https://cloud.google.com/dataproc/docs/guides/create-cluster)
- [Cloud Storage バケットを作成する](https://cloud.google.com/storage/docs/creating-buckets)
- [PySpark DataFrame 構文](https://spark.apache.org/docs/latest/api/python/reference/pyspark.sql/api/pyspark.sql.DataFrame.html)

</TabItem>

</Tabs>

