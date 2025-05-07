---
datatype: directorypath
default_value: dbt_packages
---

<File name='dbt_project.yml'>

```yml
packages-install-path: directorypath
```

</File>

## 定義
オプションで、`dbt deps` [コマンド](/reference/commands/deps) を実行する際に [パッケージ](/docs/build/packages) がインストールされるカスタムディレクトリを指定します。このディレクトリは通常、git によって無視されることに注意してください。

## デフォルト
デフォルトでは、dbt はパッケージを `dbt_packages` ディレクトリにインストールします。つまり、`packages-install-path: dbt_packages` です。

## 例
### `dbt_packages` ではなく `packages` という名前のサブディレクトリにパッケージをインストールします

<File name='dbt_project.yml'>

```yml
packages-install-path: packages
```

</File>
