---
title: "ソースからインストール"
description: "You can install dbt Core from its GitHub code source."
pagination_next: null
---

dbt Core とそのアダプタ プラグインのほぼすべてはオープン ソース ソフトウェアです。そのため、コードベースはソースからダウンロードしてビルドすることができます。最新のコードが必要な場合や、特定のコミットから dbt をインストールする場合は、ソースからインストールできます。これは、変更をコントリビュートする場合や、過去の変更をデバッグする場合に役立ちます。

ソースからダウンロードするには、GitHub からリポジトリをクローンしてローカル コピーを作成し、`pip` を使用してローカル バージョンをインストールします。

dbt Core をダウンロードしてビルドすると、バグを修正したり、求められている機能を実装したりして、プロジェクトに貢献できます。詳細については、[コントリビュート ガイドライン](https://github.com/dbt-labs/dbt-core/blob/HEAD/CONTRIBUTING.md) をお読みください。

### dbt Core のインストール

v1.8以降、アダプタをインストールしても`dbt-core`は自動的にインストールされません。これは、アダプタとdbt Coreバージョンが互いに分離されているため、既存のdbt-coreインストールを上書きしたくないためです。

<VersionBlock firstVersion="1.8">

GitHub コード ソースからのみ `dbt-core` をインストールするには:

```shell
git clone https://github.com/dbt-labs/dbt-core.git
cd dbt-core
python -m pip install -r requirements.txt
```

</VersionBlock>

<VersionBlock lastVersion="1.7">

To install `dbt-core` and `dbt-postgres` from the GitHub code source:

```shell
git clone https://github.com/dbt-labs/dbt-core.git
cd dbt-core
python -m pip install -r requirements.txt
```
</VersionBlock>

ローカルで行った変更が編集可能モードでインストールするには、次の手順を実行します:

```shell
python -m pip install -e editable-requirements.txt` 
```
instead.

### アダプタプラグインのインストール

ソースからアダプタ プラグインをインストールするには、まずそのソース リポジトリを見つける必要があります。たとえば、`dbt-redshift` アダプタは https://github.com/dbt-labs/dbt-redshift.git にあるので、そこからクローンしてインストールできます:

<VersionBlock firstVersion="1.8">

アダプタ プラグインをインストールする前に、`dbt-core` もインストールする必要があります。

</VersionBlock>

<VersionBlock lastVersion="1.7">

You do _not_ need to install `dbt-core` before installing an adapter plugin -- the plugin includes `dbt-core` among its dependencies, and it will install the latest compatible version automatically.
</VersionBlock>

```shell
git clone https://github.com/dbt-labs/dbt-redshift.git
cd dbt-redshift
python -m pip install .
```

貢献中など編集可能なモードでインストールするには、代わりに `python -m pip install -e .` を使用します。

<FAQ path="Core/install-pip-os-prereqs" />
<FAQ path="Core/install-python-compatibility" />
<FAQ path="Core/install-pip-best-practices" />
