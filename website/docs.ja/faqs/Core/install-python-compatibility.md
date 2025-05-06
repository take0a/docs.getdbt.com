---
title: どのバージョンの Python を使用できますか?
description: "dbt Core でサポートされている Python バージョン"
sidebar_label: 'Python version'
id: install-python-compatibility
---

import Pythonmatrix from '/snippets/_python-compatibility-matrix.md';

この表を使用して、dbt-core のバージョンと互換性のある Python のバージョンを照合してください。新しい [dbt マイナーバージョン](/docs/dbt-versions/core#minor-versions) では、すべての依存関係がサポートできる場合、新しい Python3 マイナーバージョンのサポートが追加されます。また、dbt マイナーバージョンでは、古い Python3 マイナーバージョンのサポートが [サポート終了](https://endoflife.date/python) 前に終了します。

<Pythonmatrix/>

アダプタプラグインとその依存関係は、必ずしも最新バージョンのPythonと互換性があるとは限りません。

[dbt Pythonモデル](/docs/build/python-models#specific-data-platforms)と混同しないでください。Snowparkをサポートするデータプラットフォームを使用している場合は、`python_version`設定を使用して、[Pythonバージョン](https://docs.snowflake.com/en/developer-guide/snowpark/python/setup) 3.9、3.10、または3.11でSnowparkモデルを実行してください。
