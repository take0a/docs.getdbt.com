---
title: "サポートされているデータプラットフォーム"
id: "supported-data-platforms"
sidebar_label: "About supported data platforms"
description: "Connect dbt to any data platform in dbt Cloud or dbt Core, using a dedicated adapter plugin"
hide_table_of_contents: true
pagination_next: "docs/connect-adapters"
pagination_prev: null
---

dbt は、データベース、ウェアハウス、レイク、またはクエリ エンジンに接続し、SQL を実行します。これらの SQL 対応プラットフォームは、まとめて _データ プラットフォーム_ と呼ばれます。dbt は、それぞれ専用のアダプタ プラグインを使用してデータ プラットフォームに接続します。プラグインは Python モジュールとして構築され、システムにインストールされている場合は dbt Core によって検出されます。詳細については、[アダプタの構築、テスト、ドキュメント化、およびプロモート](/guides/adapter-creation) ガイドを参照してください。

dbt Cloud でアダプタとデータ プラットフォームにネイティブに [接続](/docs/connect-adapters) することも、dbt Core を使用して手動でインストールすることもできます。

構成によって、特定のデータ プラットフォームでの dbt の動作をさらにカスタマイズすることもできます。例については、[Postgres の構成](/reference/resource-configs/postgres-configs) を参照してください。

## アダプタの種類

現在、2 種類のアダプタが利用可能です:

- **信頼済み** - [信頼済みアダプタ](trusted-adapters) は、アダプタの保守担当者が Trusted Adapter Program に参加することを決定し、その要件を満たすことを約束しているアダプタです。dbt Cloud でサポートされているアダプタの場合、保守担当者は、開発、ドキュメント、ユーザー エクスペリエンス、およびメンテナンスに関する契約上の要件をカバーする追加の厳格なプロセスを経ています。
- **コミュニティ** - [コミュニティ アダプタ](community-adapters) はオープン ソースであり、コミュニティ メンバーによって保守されています。これらのアダプタは Trusted Adapter Program の一部ではなく、使用上の不一致が生じる可能性があります。

<details>
  <summary>オープンソース プロジェクトに依存する場合の考慮事項</summary>

  1. 動作しますか?
  2. コードを「所有」している人はいますか、または動作を保証する責任がある人はいますか?
  3. バグはすぐに修正されますか?
  4. 新しい dbt Core 機能で最新の状態に保たれていますか?
  5. 使用量は自立できるほど十分ですか?
  6. 他の既知のプロジェクトがこのライブラリに依存していますか?

</details>
