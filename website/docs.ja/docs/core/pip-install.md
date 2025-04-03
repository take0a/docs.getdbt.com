---
title: "pip でインストール"
description: "Install dbt Core and adapter plugins from the command line with pip."
---

Windows、Linux、または MacOS オペレーティング システムに dbt Core をインストールするには、`pip` を使用する必要があります。

dbt Core とプラグインは [PyPI](https://pypi.org/project/dbt-core/) で配布されている Python モジュールであるため、`pip` を使用してインストールできます。

<FAQ path="Core/install-pip-os-prereqs" />
<FAQ path="Core/install-python-compatibility" />

## Python 仮想環境とは何ですか？

Python 仮想環境は、Python プロジェクト用の分離されたワークスペースを作成し、異なるプロジェクトやバージョンの依存関係間の競合を防ぎます。

[conda](https://anaconda.org/anaconda/conda)、[poetry](https://python-poetry.org/docs/managing-environments/)、`venv` などのツールを使用して仮想環境を作成できます。このガイドでは、軽量で追加の依存関係が最も少なく、デフォルトで Python に含まれている `venv` を使用します。

[dbt Core](/docs/core/installation-overview) や [dbt Cloud CLI](/docs/cloud/cloud-cli-installation#install-a-virtual-environment) などで dbt をローカルで実行したいユーザーは、Python 仮想環境をインストールすることをお勧めします。

### 前提条件

- ターミナルまたはコマンド プロンプトにアクセスできる。
- マシンに [Python](https://www.python.org/downloads/) がインストールされている。ターミナルまたはコマンド プロンプトで `python --version` または `python3 --version` を実行すると、Python がインストールされているかどうかを確認できます。
- [pip](https://pip.pypa.io/en/stable/installation/) がインストールされている。`pip --version` または `pip3 --version` を実行すると、pip がインストールされているかどうかを確認できます。
- マシンにディレクトリを作成し、パッケージをインストールするために必要な権限を持っている。
- 前提条件を満たしたら、次の手順に従って仮想環境を設定します。

### Python 仮想環境を設定する

`venv` は、`env` フォルダ内に Python 仮想環境を設定します。

使用するオペレーティング システムに応じて、仮想環境を設定するための特定の手順を実行する必要があります。

Python 仮想環境を設定するには、プロジェクト ディレクトリに移動してコマンドを実行します。これにより、任意の名前を付けることができるローカル フォルダ内に新しい仮想環境が生成されます。[私たちの慣例](https://github.com/dbt-labs/dbt-core/blob/main/CONTRIBUTING.md#virtual-environments) では、`env` または `env-anything-you-want` という名前を付けています。

<Tabs>
  <TabItem value="Unix/macOS" label="Unix/macOS">
    1. 仮想環境を作成します:

    ```shell
    python3 -m venv env
    ```

    2. 仮想環境をアクティブ化します:

    ```shell
    source env/bin/activate
    ```

    3. Python パスを確認します:

    ```shell
    which python
    ```

    4. Python を実行します:

    ```shell
    env/bin/python
    ```
  </TabItem>

  <TabItem value="Windows" label="Windows">
    1. 仮想環境を作成します:

    ```shell
    py -m venv env
    ```

    2. 仮想環境をアクティブ化します:

    ```shell
    env\Scripts\activate
    ```

    3. Python パスを確認します:

    ```shell
    where python
    ```

    4. Python を実行します:

    ```shell
    env\Scripts\python
    ```
  </TabItem>
</Tabs>

dbt Core を使用している場合は、仮想環境を作成した後、[pip を使用して dbt Core をインストールするためのベスト プラクティスは何ですか?](/faqs/Core/install-pip-best-practices.md#using-virtual-environments) を参照してください。

dbt Cloud CLI を使用している場合は、仮想環境を作成した後、[pip で dbt Cloud CLI をインストール](/docs/cloud/cloud-cli-installation#install-dbt-cloud-cli-in-pip) できます。

### 仮想環境を非アクティブ化する

プロジェクトを切り替えたり、仮想環境を離れたりするには、仮想環境がアクティブなときに次のコマンドを使用して環境を非アクティブ化します:

```shell
deactivate
```

### エイリアスを作成する

新しいシェル ウィンドウまたはセッションごとに dbt 環境をアクティブ化するには、`$HOME/.bashrc`、`$HOME/.zshrc`、またはシェルが取得する構成ファイルにソース コマンドのエイリアスを作成します。

たとえば、rc ファイルに次のコードを追加し、`<PATH_TO_VIRTUAL_ENV_CONFIG>` を仮想環境構成へのパスに置き換えます。

```shell
alias env_dbt='source <PATH_TO_VIRTUAL_ENV_CONFIG>/bin/activate'
```

## アダプタのインストール

使用する [アダプタ](/docs/supported-data-platforms) を決定したら、コマンド ラインを使用してインストールできます。v1.8 以降では、アダプタをインストールしても `dbt-core` は自動的にインストールされません。これは、アダプタと dbt Core バージョンが互いに分離され、既存の dbt-core インストールを上書きしないようにしたためです。

<VersionBlock firstVersion="1.8">

```shell
python -m pip install dbt-core dbt-ADAPTER_NAME
```

</VersionBlock>

<VersionBlock lastVersion="1.7">

```shell
python -m pip install dbt-ADAPTER_NAME
```

</VersionBlock>

たとえば、Postgres を使用する場合:

<VersionBlock firstVersion="1.8">

```shell
python -m pip install dbt-core dbt-postgres
```

これにより、`dbt-core` と `dbt-postgres` のみがインストールされます:

```shell
$ dbt --version
installed version: 1.0.0
   latest version: 1.0.0

Up to date!

Plugins:
  - postgres: 1.0.0
```

すべてのアダプタは `dbt-core` 上に構築されます。一部のアダプタは他のアダプタにも依存します。たとえば、`dbt-redshift` は `dbt-postgres` 上に構築されます。その場合、特定のインストールにそれらのアダプタも含まれることになります。
</VersionBlock>

<VersionBlock lastVersion="1.7">

```shell
python -m pip install dbt-postgres
```

これにより、`dbt-core` と `dbt-postgres` のみがインストールされます:

```shell
$ dbt --version
installed version: 1.0.0
   latest version: 1.0.0

Up to date!

Plugins:
  - postgres: 1.0.0
```

一部のアダプタは他のアダプタに依存します。たとえば、`dbt-redshift` は `dbt-postgres` の上に構築されます。その場合、特定のインストールにそれらのアダプタも含まれることになります。
</VersionBlock>

### アダプタのアップグレード

特定のアダプタ プラグインをアップグレードするには:

```shell
python -m pip install --upgrade dbt-ADAPTER_NAME
```

### dbt-core のみをインストールする

dbt Core と統合するツールを構築する場合は、データベース アダプターなしでコア ライブラリのみをインストールすることをお勧めします。dbt を CLI ツールとして使用することはできないことに注意してください。

```shell
python -m pip install dbt-core
```

## dbt Core のバージョンを変更する

コマンドライン (CLI) で `--upgrade` オプションを使用すると、dbt Core のバージョンをアップグレードまたはダウングレードできます。詳細については、[Core バージョンのアップグレードに関するベスト プラクティス](/docs/dbt-versions/core#best-practices-for-upgrading) を参照してください。

dbt を最新バージョンにアップグレードするには:

```
python -m pip install --upgrade dbt-core
```

古いバージョンにダウングレードするには、使用するバージョンを指定します。このコマンドは、パッケージの依存関係を解決するときに役立ちます。例:

```
python -m pip install --upgrade dbt-core==1.9
```

## `pip install dbt`

2023 年秋、PyPI の `dbt` パッケージは、[dbt Cloud CLI](/docs/cloud/cloud-cli-installation?install=pip#install-dbt-cloud-cli-in-pip) をインストールするためのサポート対象メソッドになりました。

`dbt` という名前のパッケージのインストールに依存するワークフローまたは統合がある場合は、そのパッケージで使用されたのと同じ 5 つのパッケージをインストールすることで、同じ動作を実現できます:

```shell
python -m pip install \
  dbt-core \
  dbt-postgres \
  dbt-redshift \
  dbt-snowflake \
  dbt-bigquery \
  dbt-trino
```

あるいは、必要なパッケージだけをインストールするのも良いでしょう。

<VersionBlock firstVersion="1.8">

## プレリリースのインストール

プレリリース アダプターは、最終的な安定バージョンより前にリリースされるバージョンです。これにより、ユーザーは新機能をテストし、フィードバックを提供し、今後の機能に早期にアクセスして、システムが最終リリースに対応できるようにすることができます。

アダプターのプレリリースを使用すると、安定リリースに先立って新機能や改善点に早期にアクセスできるなど、多くの利点があります。互換性テストだけでなく、自分の環境でアダプターをテストして統合の問題を早期に把握し、システムが最終リリースに対応できるようにすることができます。

最終的な安定バージョンより前にプレリリース バージョンを使用すると、バージョンが完全に最適化されていないため、予期しない動作が発生する可能性があります。さらに、プレリリース フェーズ中に頻繁に更新やパッチを適用すると、メンテナンスに余分な時間と労力が必要になる場合があります。さらに、`--pre フラグ` により、他の依存関係の互換性のあるプレリリース バージョンがインストールされる可能性があり、不安定さが増す可能性があります。

dbt Core とアダプターのプレリリース バージョンをインストールするには、このコマンドを使用します (`dbt-adapter-name` をアダプターに置き換えます)

```shell
python3 -m pip install --pre dbt-core dbt-adapter-name
```

たとえば、Snowflake を使用している場合は、次のコマンドを使用します:


```shell
python3 -m pip install --pre dbt-core dbt-snowflake

```

プレリリースは [仮想 Python 環境](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/) にインストールすることをお勧めします。たとえば、プレリリースを `POSIX bash`/`zsh` 仮想 Python 環境にインストールするには、次のコマンドを使用します:

```shell
dbt --version
python3 -m venv .venv
source .venv/bin/activate
python3 -m pip install --upgrade pip
python3 -m pip install --pre dbt-core dbt-adapter-name
source .venv/bin/activate
dbt --version
```
注意: これにより、すべての依存関係のプレリリースもインストールされます。

### 仮想環境をアクティブ化する

仮想環境内でパッケージをインストールまたは使用するには:

- 仮想環境をアクティブ化して、特定の Python および `pip` 実行可能ファイルをシェルの PATH に追加します。これにより、環境の分離されたセットアップが確実に使用されます。

詳細については、[仮想環境の作成と使用](https://packaging.python.org/en/latest/guides/installing-using-pip-and-virtual-environments/#create-and-use-virtual-environments) を参照してください。

オペレーティング システムを選択し、次のコマンドを実行してアクティブ化します:

<Expandable alt_header="Unix/macOS" >

1. 仮想環境をアクティブ化します:

```shell
source .venv/bin/activate
which python
.venv/bin/python
  
```
  2. 次のコマンドを使用してプレリリースをインストールします:


```shell
python3 -m pip install --pre dbt-core dbt-adapter-name
source .venv/bin/activate
dbt --version
```

</Expandable>

<Expandable alt_header="Windows" >

1. 仮想環境をアクティブ化します:

```shell
.venv\Scripts\activate
where python
.venv\Scripts\python
```

2. 次のコマンドを使用してプレリリースをインストールします:

```shell
py -m pip install --pre dbt-core dbt-adapter-name
.venv\Scripts\activate
dbt --version
```

</Expandable>


</VersionBlock>

<VersionBlock lastVersion="1.7">

### Installing prereleases

`dbt-adapters` is only compatible with dbt Core 1.8 and higher. If you're on dbt Core v1.7 or lower, follow these steps to upgrade to v1.8 or higher to install prereleases of `dbt-adapters`.

```shell
python -m pip uninstall -y dbt-adapters
python -m pip install --upgrade --pre dbt-core dbt-common dbt-adapters
dbt --version
```

</VersionBlock>