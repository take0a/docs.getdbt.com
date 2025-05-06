---
title: "オペレーティング システムには前提条件がありますか?"
description: "dbt Core をインストールするための前提条件がオペレーティング システムにあるかどうかを確認できます。"
sidebar_label: 'dbt Core システムの前提条件'
id: install-pip-os-prereqs.md

---

お使いのオペレーティング システムによっては、pip を使用して dbt Core をインストールする前に事前設定が必要な場合があります。開発環境固有の依存関係をダウンロードしてインストールしたら、[dbt Core の pip インストール](/docs/core/pip-install) に進むことができます。

### CentOS

CentOS では、dbt Core を正常にインストールして実行するには、Python とその他の依存関係が必要です。

Python とその他の依存関係をインストールするには、以下の手順に従います:

```shell

sudo yum install redhat-rpm-config gcc libffi-devel \
  python-devel openssl-devel

```

### MacOS

MacOS で dbt Core を正常にインストールして実行するには、Python 3.8 以降が必要です。

Python のバージョンを確認するには:

```shell

python --version

```

互換性のあるバージョンが必要な場合は、[MacOS 用 Python バージョン 3.9 以上](https://www.python.org/downloads/macos) をダウンロードしてインストールしてください。

お使いのマシンが Apple M1 アーキテクチャで動作している場合は、[Rosetta](https://support.apple.com/en-us/HT211861) 経由で dbt をインストールすることをお勧めします。これは、Intel プロセッサでのみサポートされている特定の依存関係に必要なためです。

### Ubuntu/Debian

Ubuntu で dbt Core を正常にインストールして実行するには、Python とその他の依存関係が必要です。

Python とその他の依存関係をインストールするには、以下の手順に従います:

```shell

sudo apt-get install git libpq-dev python-dev python3-pip
sudo apt-get remove python-cffi
sudo pip install --upgrade cffi
pip install cryptography~=3.4

```

### Windows

Windows で dbt Core を正常にインストールして実行するには、Python と Git が必要です。

[Git for Windows](https://git-scm.com/downloads) と [Python バージョン 3.9 以上 (Windows 用)](https://www.python.org/downloads/windows/) をインストールしてください。

その他のご質問については、[Python 互換性に関する FAQ](/faqs/Core/install-python-compatibility) をご覧ください。
