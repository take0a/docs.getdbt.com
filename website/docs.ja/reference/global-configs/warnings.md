---
title: "Warnings"
id: "warnings"
sidebar: "Warnings"
---

`WARN_ERROR` 設定を有効にすると、dbt の警告がエラーに変換されます。dbt が通常警告を出力する箇所では、エラーが発生します。例としては、リソースを全く選択しない `--select` 条件、非推奨、モデルが関連付けられていない設定、無効なテスト設定、警告を返すように設定されているテストやフレッシュネスチェックなどが挙げられます。

<File name='Usage'>

```text
dbt --warn-error run
...
```

</File>


警告をエラーに変換することはニーズに完全に合致するかもしれませんが、中にはそれほど重要でない警告もあれば、非常に重要な警告もあるかもしれません。`WARN_ERROR_OPTIONS` 設定を使用すると、_どの種類の警告_をエラーとして扱うかをより細かく制御できます。

<VersionBlock lastVersion="1.7">

Warnings that should be treated as errors can be specified through `include` and/or `exclude` parameters. Warning names can be found in [dbt-core's types.py file](https://github.com/dbt-labs/dbt-core/blob/main/core/dbt/events/types.py), where each class name that inherits from `WarnLevel` corresponds to a warning name (e.g. `AdapterDeprecationWarning`, `NoNodesForSelectionCriteria`).

The `include` parameter can be set to `"all"` or `"*"` to treat all warnings as exceptions, or to a list of specific warning names to treat as exceptions. When `include` is set to `"all"` or `"*"`, the optional `exclude` parameter can be set to exclude specific warnings from being treated as exceptions.

</VersionBlock>

<VersionBlock firstVersion="1.8">

- エラーとして扱うべき警告は、`error` および `warn` パラメータで指定できます。警告名は [dbt-core の types.py ファイル](https://github.com/dbt-labs/dbt-core/blob/main/core/dbt/events/types.py) に記載されています。`WarnLevel` を継承する各クラス名は、警告名に対応しています (例: `AdapterDeprecationWarning`、`NoNodesForSelectionCriteria`)。

- `error` パラメータを `"all"` または `"*"` に設定すると、すべての警告を例外として扱うことができます。また、例外として扱う特定の警告名のリストを設定することもできます。`error` を `"all"` または `"*"` に設定すると、オプションの `warn` パラメータを設定することで、特定の警告を例外として扱わないようにすることができます。

- `silence` パラメータを使用すると、プロジェクトフラグを通じて警告を無視できます。毎回、警告リストを再指定する必要はありません。例えば、非推奨の警告やプロジェクト全体で無視したい特定の警告を `silence` パラメータで指定できます。これは、大規模プロジェクトで特定の警告が重要ではなく、ノイズを抑えてログをクリーンに保つために無視できる場合に便利です。


<File name='dbt_project.yml'>

```yaml
name: "my_dbt_project"
tests:
  +enabled: True
flags:
  warn_error_options:
    error: # Previously called "include"
    warn: # Previously called "exclude"
    silence: # To silence or ignore warnings
      - NoNodesForSelectionCriteria
```

</File>

</VersionBlock>


:::info `WARN_ERROR` と `WARN_ERROR_OPTIONS` は相互に排他的です。
`WARN_ERROR` と `WARN_ERROR_OPTIONS` は相互に排他的です。複数の場所（例：環境変数 + CLI フラグ）で設定を指定する場合でも、どちらか一方しか指定できません。それ以外の場合は、使用方法エラーが表示されます。
:::

<VersionBlock lastVersion="1.7">

```text
dbt --warn-error-options '{"include": "all"}' run
...
```

```text
dbt --warn-error-options '{"include": "all", "exclude": ["NoNodesForSelectionCriteria"]}' run
...
```


```text
dbt --warn-error-options '{"include": ["NoNodesForSelectionCriteria"]}' run
...
```

```text
DBT_WARN_ERROR_OPTIONS='{"include": ["NoNodesForSelectionCriteria"]}' dbt run
...
```

Values for `error`, `warn`, and/or `silence` should be passed on as arrays. For example, `dbt --warn-error-options '{"error": "all", "warn": ["NoNodesForSelectionCriteria"]}' run` not `dbt --warn-error-options '{"error": "all", "warn": "NoNodesForSelectionCriteria"}' run`.


<File name='profiles.yml'>

```yaml
config:
  warn_error_options:
    include: all
    exclude: 
      - NoNodesForSelectionCriteria
```

</File>

</VersionBlock>

<VersionBlock firstVersion="1.8">

```text
dbt --warn-error-options '{"error": "all"}' run
...
```

```text
dbt --warn-error-options '{"error": "all", "warn": ["NoNodesForSelectionCriteria"]}' run
...
```


```text
dbt --warn-error-options '{"error": ["NoNodesForSelectionCriteria"]}' run
...
```

```text
DBT_WARN_ERROR_OPTIONS='{"error": ["NoNodesForSelectionCriteria"]}' dbt run
...
```

<File name='profiles.yml'>

```yaml
config:
  warn_error_options:
    error: # Previously called "include"
    warn: # Previously called "exclude"
      - NoNodesForSelectionCriteria
    silence: # Silence or ignore warnings
      - NoNodesForSelectionCriteria
```

</File>

</VersionBlock>
