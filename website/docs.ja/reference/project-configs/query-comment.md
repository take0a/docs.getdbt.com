---
datatype: "string | {comment: string, append: true | false }"
default: >
  /* {"app": "dbt", "dbt_version": "1.5.0rc2", "profile_name": "debug", "target_name": "dev", "node_id": "model.dbt2.my_model"} */
---

<File name='dbt_project.yml'>

```yml
query-comment: string
```

</File>

`query-comment` 構成では、次のように辞書入力も受け入れます:

<File name='dbt_project.yml'>

```yml
models:
  my_dbt_project:
    +materialized: table

query-comment:
  comment: string
  append: true | false
  job-label: true | false  # BigQuery only
```

</File>

## 定義
dbt がデータベースに対して実行する各クエリにコメントとして挿入する文字列。このコメントにより、SQL 文をモデルやテストなどの特定の dbt リソースに関連付けることができます。

`query-comment` 設定では、文字列を返すマクロを呼び出すこともできます。

## デフォルト
デフォルトでは、dbt はクエリの先頭に <Term id="json" /> コメントを挿入します。このコメントには、dbt のバージョン、プロファイル名とターゲット名、実行するリソースのノード ID などの情報が含まれます。例:

```sql
/* {"app": "dbt", "dbt_version": "1.5.0rc2", "profile_name": "debug",
    "target_name": "dev", "node_id": "model.dbt2.my_model"} */

create view analytics.analytics.orders as (
    select ...
  );
```




## 辞書構文の使用
辞書構文には2つのキーが含まれます。
* `comment` (オプション、詳細については[default](#default)セクションを参照してください): クエリにコメントとして挿入する文字列。
* `append` (オプション、デフォルト=`false`): コメントを追加するか(クエリの末尾に追加するか)、追加しないか(クエリの先頭に追加するか)を指定します。デフォルトでは、コメントはクエリの先頭に追加されます(`append: false`)。

この構文は、Snowflakeなどの[先頭のSQLコメントを削除する](https://docs.snowflake.com/en/release-notes/2017-04.html#queries-leading-comments-removed-during-execution)データベースで役立ちます。

## 例

### 静的コメントを先頭に追加する
次の例では、dbt が実行する SQL クエリのヘッダーに `/* executed by dbt */` というコメントを挿入します。

<File name='dbt_project.yml'>

```yml
query-comment: "executed by dbt"

```

</File>

**出力例:**

```sql
/* executed by dbt */

select ...
```

### クエリコメントを無効にする

<File name='dbt_project.yml'>

```yml
query-comment:

```

</File>

または：

<File name='dbt_project.yml'>

```yml
query-comment: null

```

</File>

### 動的なコメントを先頭に追加する
次の例では、アクティブな dbt ターゲットで指定された構成済みの「user」に基づいて変化するコメントを挿入します。
<File name='dbt_project.yml'>

```yml
query-comment: "run by {{ target.user }} in dbt"

```

</File>

**出力例:**

```sql
/* run by drew in dbt */

select ...
```

### デフォルトコメントを追加する
次の例では、辞書構文を使用して、デフォルトコメントを（先頭ではなく）末尾に追加します。

デフォルトコメントを追加できるようにするために、`comment:` フィールドが省略されていることに注意してください。

<File name='dbt_project.yml'>

```yaml

query-comment:
  append: True
```

</File>

**出力例:**

```sql
select ...
/* {"app": "dbt", "dbt_version": "1.6.0rc2", "profile_name": "debug", "target_name": "dev", "node_id": "model.dbt2.my_model"} */
;
```

### BigQuery: クエリコメント項目をジョブラベルとして含める

`query-comment.job-label` が true に設定されている場合、dbt はクエリコメント項目（辞書形式の場合）またはコメント文字列を、実行するクエリのジョブラベルとして含めます。これらのラベルは、[BigQuery 固有の設定](/reference/project-configs/query-comment#bigquery-include-query-comment-items-as-job-labels) で指定されたラベルに加えて含められます。

<File name='dbt_project.yml'>

```yaml

query-comment:
  job-label: True
```

</File>

### カスタムコメントを追加する
次の例では、辞書構文を使用して、アクティブな dbt ターゲットで指定された構成済みの「ユーザー」に基づいて変化するコメントを（先頭ではなく）末尾に追加します。

<File name='dbt_project.yml'>

```yaml

query-comment:
  comment: "run by {{ target.user }} in dbt"
  append: True
```

</File>

**出力例:**

```sql
select ...
/* run by drew in dbt */
;
```



### 中級：マクロを使ってコメントを生成する

`query-comment` 設定は、dbt プロジェクト内のマクロを参照できます。`macros` ディレクトリに任意の名前（`query_comment` などが良いでしょう）のマクロを以下のように作成するだけです。

<File name='macros/query_comment.sql'>

```jinja2

{% macro query_comment() %}

  dbt {{ dbt_version }}: running {{ node.unique_id }} for target {{ target.name }}

{% endmacro %}
```

</File>

次に、`dbt_project.yml` ファイルでマクロを呼び出します。YAML パーサーが `{` を辞書の開始として解釈しないように、マクロを引用符で囲んでください。

<File name='dbt_project.yml'>

```yaml
query-comment: "{{ query_comment() }}"

```

</File>

### 上級：マクロを使用してコメントを生成する

次の例は、解析することで dbt プロジェクトのパフォーマンス特性を把握できる JSON クエリコメントを示しています。

<File name='macros/query_comment.sql'>

```jinja2
{% macro query_comment(node) %}
    {%- set comment_dict = {} -%}
    {%- do comment_dict.update(
        app='dbt',
        dbt_version=dbt_version,
        profile_name=target.get('profile_name'),
        target_name=target.get('target_name'),
    ) -%}
    {%- if node is not none -%}
      {%- do comment_dict.update(
        file=node.original_file_path,
        node_id=node.unique_id,
        node_name=node.name,
        resource_type=node.resource_type,
        package_name=node.package_name,
        relation={
            "database": node.database,
            "schema": node.schema,
            "identifier": node.identifier
        }
      ) -%}
    {% else %}
      {%- do comment_dict.update(node_id='internal') -%}
    {%- endif -%}
    {% do return(tojson(comment_dict)) %}
{% endmacro %}
```

</File>

上記のマクロを以下のように呼び出します。


<File name='dbt_project.yml'>

```yaml
query-comment: "{{ query_comment(node) }}"

```

</File>

## コンパイルコンテキスト

クエリコメントを生成する際に、以下のコンテキスト変数が利用できます。

| Context Variable | Description |
| ---------------- | ----------- |
| dbt_version      | 使用されているdbtのバージョン。リリースバージョン管理の詳細については、[バージョン管理](/reference/commands/version#versioning)を参照してください。 |
| env_var          | See [env_var](/reference/dbt-jinja-functions/env_var) |
| modules          | See [modules](/reference/dbt-jinja-functions/modules) |
| run_started_at   | dbt呼び出しが開始されたとき |
| invocation_id    | dbt呼び出しの一意のID |
| fromjson         | See [fromjson](/reference/dbt-jinja-functions/fromjson) |
| tojson           | See [tojson](/reference/dbt-jinja-functions/tojson) |
| log              | See [log](/reference/dbt-jinja-functions/log) |
| var              | See [var](/reference/dbt-jinja-functions/var) |
| target           | See [target](/reference/dbt-jinja-functions/target) |
| connection_name  | 接続の内部名を表す文字列。この文字列はdbtによって生成されます。 |
| node             | 解析されたノードオブジェクトの辞書表現。`node.unique_id`、`node.database`、`node.schema` などを使用します。 |

注: `query-comment` マクロの `var()` 関数は、CLI の `--vars` 引数で渡された変数にのみアクセスします。`dbt_project.yml` の vars ブロックで定義された変数は、クエリコメントの生成時にはアクセスできません。
