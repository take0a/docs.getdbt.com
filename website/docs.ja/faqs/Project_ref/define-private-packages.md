---
title: dependencies.yml ファイルでプライベート パッケージを定義できますか?
sidebar_label: プライベートパッケージを定義する
id: define-private-packages
description: プロジェクトでプライベートパッケージを定義する方法を学びます
---

プライベートパッケージへのアクセス方法によって異なります:

- [ネイティブプライベートパッケージ](/docs/build/packages#native-private-packages)を使用している場合は、`dependencies.yml`ファイルで定義できます。
- [git token method](/docs/build/packages#git-token-method)を使用している場合は、`dependencies.yml`ファイルではなく、`packages.yml`ファイルで定義する必要があります。これは、`dependencies.yml`では条件付きレンダリング（Jinja-in-yamlなど）がサポートされていないためです。
