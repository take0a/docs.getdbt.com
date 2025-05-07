---
datatype: version-range | [version-range]
description: "dbt の require-dbt-version 構成を理解するには、このガイドをお読みください。"
default_value: None
---



<File name='dbt_project.yml'>

```yml
require-dbt-version: version-range | [version-range]
```

</File>

## 定義

`require-dbt-version` を使用すると、プロジェクトが特定のバージョンの dbt でのみ動作するように制限できます。

この設定を行うと、サポートされていないバージョンの dbt を使用してプロジェクトを実行しようとしたユーザーに、dbt は役立つエラーメッセージを送信します。これは、パッケージメンテナー（[dbt-utils](https://github.com/dbt-labs/dbt-utils) など）が、ユーザーの dbt バージョンがパッケージと互換性があることを確認するのに役立ちます。また、この設定を行うことで、ローカル開発においてチーム全体が同じバージョンの dbt で同期を保つことができ、動作の変更による互換性の問題を回避できます。

この設定が指定されていない場合、バージョンチェックは行われません。

:::info dbt Cloud release tracks 

<Snippet path="_config-dbt-version-check" />

:::

### YAML の引用符で囲む

この設定は、YAML パーサーによって文字列として展開される必要があります。そのため、設定の値は引用符で囲み、空白文字を避けるように注意してください。例:

```yml
# ✅ These will work
require-dbt-version: ">=1.0.0" # Double quotes are OK
require-dbt-version: '>=1.0.0' # So are single quotes

# ❌ These will not work
require-dbt-version: >=1.0.0 # No quotes? No good
require-dbt-version: ">= 1.0.0" # Don't put whitespace after the equality signs
```


## 例

### 最小の dbt バージョンを指定します
最小境界には `>=` 演算子を使用します。次の例では、このプロジェクトは dbt のバージョン 1.0.0 以上で実行されます。


<File name='dbt_project.yml'>

```yml
require-dbt-version: ">=1.0.0"
```

</File>


### 範囲に固定する
上限と下限をカンマ区切りのリストで指定します。以下の例では、このプロジェクトは dbt 1.x.x で実行されます。

<File name='dbt_project.yml'>

```yml
require-dbt-version: [">=1.0.0", "<2.0.0"]
```

</File>

または

<File name='dbt_project.yml'>

```yml
require-dbt-version: ">=1.0.0,<2.0.0"
```

</File>

  
### 特定のdbtバージョンを要求する

:::info 非推奨
特定の dbt バージョンへの固定は、プロジェクトの柔軟性が制限され、特に dbt パッケージで互換性の問題が発生する可能性があるため、推奨されません。互換性を高め、アップデートのメリットを享受するには、バージョン範囲（例：`">=1.0.0"、"<2.0.0"`）を使用して[メジャーリリースに固定](#範囲で固定)することをお勧めします。

プロジェクトを特定のバージョンの dbt Core でのみ実行するように制限することは可能ですが、dbt Core v1.0.0 以降では推奨されません。
:::

次の例では、プロジェクトは dbt v1.5 でのみ実行されます:

<File name='dbt_project.yml'>

```yml
require-dbt-version: "1.5.0"
```

</File>

## 無効な dbt バージョン

プロジェクトの起動に使用された dbt のバージョンが、プロジェクトまたは含まれるパッケージのいずれかで指定された `require-dbt-version` と一致しない場合、dbt は次のエラーで直ちに失敗します:

```
$ dbt compile
Running with dbt=1.5.0
Encountered an error while reading the project:
Runtime Error
  This version of dbt is not supported with the 'my_project' package.
    Installed version of dbt: =1.5.0
    Required version of dbt for 'my_project': ['>=1.6.0', '<2.0.0']
  Check the requirements for the 'my_project' package, or run dbt again with --no-version-check
```

## バージョンチェックの無効化

互換性のない dbt バージョンによるエラーを抑制するには、`dbt run` に `--no-version-check` フラグを指定します。

```
$ dbt run --no-version-check
Running with dbt=1.5.0
Found 13 models, 2 tests, 1 archives, 0 analyses, 204 macros, 2 operations....
```

使用方法の詳細については、[グローバル設定](/reference/global-configs/version-compatibility)を参照してください。

## 推奨事項
* これは推奨設定です
* v1 より前は、必要な dbt バージョンをマイナーリリースに固定する必要があります。v1 より後は、メジャーリリースに固定する必要があります (上記の [例](#pin-to-a-range) を参照)
