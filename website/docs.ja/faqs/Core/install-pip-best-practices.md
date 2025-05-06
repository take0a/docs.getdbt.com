---
title: "pip を使用して dbt Core をインストールするためのベストプラクティスは何ですか?"
description: "pipを使ってdbt Coreをインストールする方法"
sidebar_label: 'pip で dbt Core をインストールする'
id: install-pip-best-practices.md
---

Python のローカル環境の管理は難しい場合があります。これらのベストプラクティスを活用して、pip を使用した dbt Core のインストールを改善できます。

### 仮想環境の使用

`pip` モジュールの名前空間を設定するには、[仮想環境](https://docs.python-guide.org/dev/virtualenvs/) を使用することをお勧めします。設定例を以下に示します:

```shell

python3 -m venv dbt-env				# create the environment
source dbt-env/bin/activate			# activate the environment for Mac and Linux
dbt-env\Scripts\activate			# activate the environment for Windows
```

`dbt` を仮想環境にインストールした場合、シェルウィンドウまたはセッションを作成するたびに、同じ仮想環境を再アクティブ化する必要があります。

*ヒント:* `$HOME/.bashrc`、`$HOME/.zshrc`、またはシェルが参照する rc ファイル内に `source` コマンドのエイリアスを作成できます。例えば、`alias env_dbt='source <PATH_TO_VIRTUAL_ENV_CONFIG>/bin/activate'` のようなコマンドを追加し、`<PATH_TO_VIRTUAL_ENV_CONFIG>` を仮想環境設定へのパスに置き換えます。

### 最新バージョンの使用

dbt のインストールは、最新バージョンの `pip` と `setuptools` を使用してテストされています。新しいバージョンでは、依存関係の解決に関する動作が改善されているほか、お使いのオペレーティングシステムで利用可能な場合は、プリコンパイル済みの「wheels」を使用することでインストール時間が大幅に短縮されています。

dbt をインストールする前に、最新バージョンがインストールされていることを確認してください。

```shell

python -m pip install --upgrade pip wheel setuptools

```
