---
title: dbt セマンティック レイヤーに接続しようとすると、「Failed ALPN」エラーが表示されます。
description: "dbt セマンティック レイヤーの「Failed ALPN」エラーを解決するには、dbt クラウド ドメインの SSL インターセプト例外を作成します。"
sidebar_label: 'SSL例外を使用して「Failed ALPN」エラーを解決する'
---

dbt セマンティック レイヤーを各種データ統合ツール (Tableau、DBeaver、Datagrip、ADBC、JDBC など) に接続しようとした際に「Failed ALPN」エラーが発生する場合、通常は企業 VPN またはプロキシ (Zscaler や Check Point など) の背後にあるコンピュータから接続した場合に発生します。

<Constant name="semantic_layer" />は接続に gRPC/HTTP2 を使用するため、根本原因は通常、プロキシが TLS ハンドシェイクを妨害していることです。この問題を解決するには、以下の手順を実行してください。

- プロキシが gRPC/HTTP2 をサポートしているものの、ALPN を許可するように構成されていない場合は、ALPN を許可するように設定を調整してください。または、<Constant name="cloud" /> ドメインに対して例外を作成してください。
- プロキシが gRPC/HTTP2 をサポートしていない場合は、プロキシ設定で<Constant name="cloud" /> ドメインに対して SSL インターセプトの例外を追加してください。

これにより、「Failed ALPN」エラーを回避し、接続を確立できるようになります。
