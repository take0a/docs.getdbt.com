---
datatype: [directorypath]
default_value: [target_path]
---

<File name='dbt_project.yml'>

```yml
clean-targets: [directorypath]
```

</File>


## 定義
オプションで、`dbt clean` [コマンド](/reference/commands/clean) によって削除されるディレクトリのカスタムリストを指定します。このリストには、アーティファクト（コンパイル済みファイル、ログ、インストール済みパッケージなど）を含むディレクトリのみを含める必要があります。

## デフォルト
この設定が `dbt_project.yml` ファイルに含まれていない場合、`clean` コマンドは [target-path](/reference/global-configs/json-artifacts) 内のファイルを削除します。

## 例

### `dbt clean` の一部としてパッケージとコンパイル済みファイルを削除します (推奨){#remove-packages-and-compiled-files-as-part-of-dbt-clean}


パッケージとコンパイル済みファイルを削除するには、[packages-install-path](/reference/project-configs/packages-install-path) 構成の値を `clean-targets` 構成に含めます。

<File name='dbt_project.yml'>

```yml
clean-targets:
    - target
    - dbt_packages
```

</File>

次に、`dbt clean` を実行します。

`target` ディレクトリと `dbt_packages` ディレクトリの両方が削除されます。

注: これは、`init` コマンドによって生成された dbt [スタータープロジェクト](https://github.com/dbt-labs/dbt-starter-project/blob/HEAD/dbt_project.yml) 内の設定です。


### `dbt clean` 実行時に `logs` を削除する

<File name='dbt_project.yml'>

```yml
clean-targets: [target, dbt_packages, logs]

```

</File>
