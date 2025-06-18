---
title: "External metadata ingestion"
sidebar_label: "External metadata ingestion"
description: "Connect directly to your data warehouse, giving you visibility into tables, views, and other resources that aren't defined in dbt with dbt Catalog." 
---

# External metadata ingestion <Lifecycle status="managed,managed_plus" /> <Lifecycle status="preview" />

<IntroText>

外部メタデータの取り込みを使用すると、データ ウェアハウスに直接接続して、<Constant name="explorer" /> を使用して dbt で定義されていないテーブル、ビュー、その他のリソースを可視化できます。

</IntroText>

:::info 外部メタデータの取り込みサポート
現在、外部メタデータの取り込みはSnowflakeでのみサポートされています。
:::
  
外部メタデータ認証情報を使用すると、テーブル、ビュー、コスト情報など、dbt 実行の *外部* に存在するメタデータを取り込むことができます。これらのメタデータは通常、dbt 環境がアクセスするメタデータよりも高いレベルで取り込むことができます。これは、ウェアハウス固有のインサイト（Snowflake ビューやアクセスパターンなど）で <Constant name="explorer" /> を拡充し、統合された検出エクスペリエンスを作成するのに役立ちます。

これらの認証情報は、dbt 環境の認証情報とは別に構成され、プロジェクトレベルではなくアカウントレベルでスコープが設定されます。

## 前提条件

- [Enterprise または Enterprise+](https://www.getdbt.com/pricing) プランの <Constant name="cloud" /> アカウントが必要です。
- 接続を編集するには、[権限を持つアカウント管理者](/docs/cloud/manage-access/enterprise-permissions#account-admin)である必要があります。
    - 認証情報には、[メタデータを取得するための十分な読み取りレベルのアクセス権](/docs/explore/external-metadata-ingestion#configuration-instructions)が必要です。
- [**グローバルナビゲーション**](/docs/explore/explore-projects#catalog-overview) が有効になっている必要があります。
- データプラットフォームとして Snowflake を使用してください。
- 今後のリリースにご期待ください！近日中に他のアダプターもサポートされる予定です。

## 設定手順

外部メタデータの取り込みを有効にするには：

1. [アカウント設定](/docs/cloud/account-settings) に移動します。
2. メタデータを取り込むウェアハウス接続を見つけるか、作成します。
3. **認証情報を追加** をクリックし、グローバルメタデータ認証情報を入力します。
    - これらの認証情報は、関連するデータベースとスキーマ全体でウェアハウスレベルの可視性を持っている必要があります。
4. 「外部メタデータの取り込み」オプションを有効にします。
    - これにより、この接続からのメタデータが <Constant name="explorer" /> に入力されるようになります。
    - *オプション*: **コスト最適化** などの追加機能を有効にします。
5. フィルターを適用して、取り込むメタデータを制限します。
    - **データベース**、**スキーマ**、**テーブル**、または**ビュー** でフィルタリングできます。
    - 特定のスキーマでフィルタリングすることを強くお勧めします。詳細については、[重要な考慮事項](/docs/explore/external-metadata-ingestion#important-considerations) をご覧ください。
    - 以下のフィールドはCSV形式の正規表現に対応しています。
        - 例: `DIM` は `DIM_ORDERS` および `DIMENSION_TABLE` に一致します（基本的な「contains」一致）。
        - ワイルドカードがサポートされています。`DIM*` は `DIM_ORDERS`、`DIM_PRODUCTS` などに一致します。

## 必要な認証情報

このセクションでは、Snowflake における dbt の基本的なアクセスを設定します。最小限の権限を持つロール (`dbt_metadata_role`) と、dbt のメタデータアクセス専用のユーザー (`dbt_metadata_user`) を作成します。これにより、アクセスが明確かつ制御された形で分離され、dbt はより広範な権限を必要とせずにメタデータを読み取ることができます。この設定により、dbt はプロファイリング、ドキュメント作成、リネージ作成のためのメタデータを読み取ることができますが、データの変更やリソースの管理は行えません。

1. ロールの作成:

```sql
CREATE OR REPLACE ROLE dbt_metadata_role;
```

2. メタデータを表示するためのクエリを実行するために、ウェアハウスへのアクセス権を付与します:

```sql
GRANT OPERATE, USAGE ON WAREHOUSE "<your-warehouse>" TO ROLE dbt_metadata_role;
```

ユーザーがまだいない場合は、メタデータアクセス用のdbt専用ユーザーを作成してください。`<your-password>`を強力なパスワードに、`<your-warehouse>`を上記で使用したウェアハウス名に置き換えてください:

```sql
CREATE USER dbt_metadata_user
  DISPLAY_NAME = 'dbt Metadata Integration'
  PASSWORD = 'our-password>'
  DEFAULT_ROLE = dbt_metadata_role
  TYPE = 'LEGACY_SERVICE'
  DEFAULT_WAREHOUSE = '<your-warehouse>';
```

3. ユーザーにロールを付与します:

```sql
GRANT ROLE dbt_metadata_role TO USER dbt_metadata_user;
```

注: 最小限の権限とより適切な監査のために、読み取り専用のサービス アカウントを使用します。

## メタデータアクセス権限の割り当て

このセクションでは、必要な各Snowflakeデータベースからメタデータを読み取るために必要な最小限の権限について説明します。この権限により、スキーマ、テーブル、ビュー、系統情報へのアクセスが可能になり、dbtはデータのプロファイリングとドキュメント化を実行しながら、変更を防止できます。

メタデータアクセスを許可するには、`your-database`をSnowflakeデータベースの名前に置き換えます。関連するデータベースごとにこのブロックを繰り返します。

```sql


SET db_var = '"<your-database>"';

-- Grant access to view the database and its schemas
GRANT USAGE ON DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT USAGE ON ALL SCHEMAS IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT USAGE ON FUTURE SCHEMAS IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;

-- Grant SELECT privileges to enable metadata introspection and profiling
GRANT SELECT ON ALL TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON FUTURE TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON ALL EXTERNAL TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON FUTURE EXTERNAL TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON ALL VIEWS IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON FUTURE VIEWS IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON ALL DYNAMIC TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT SELECT ON FUTURE DYNAMIC TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;

-- Grant REFERENCES to enable lineage and dependency analysis
GRANT REFERENCES ON ALL TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT REFERENCES ON FUTURE TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT REFERENCES ON ALL EXTERNAL TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT REFERENCES ON FUTURE EXTERNAL TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT REFERENCES ON ALL VIEWS IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT REFERENCES ON FUTURE VIEWS IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;

-- Grant MONITOR on dynamic tables (e.g., for freshness or status checks)
GRANT MONITOR ON ALL DYNAMIC TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;
GRANT MONITOR ON FUTURE DYNAMIC TABLES IN DATABASE IDENTIFIER($db_var) TO ROLE dbt_metadata_role;

```

## Snowflake メタデータへのアクセスを許可します

この手順では、dbt ロール (`dbt_metadata_role`) に Snowflake のシステムレベルデータベースへのアクセスを許可します。これにより、包括的なメタデータ分析に必要な使用状況統計、クエリ履歴、系統情報を読み取ることができるようになります。

Snowflake のシステムレベルデータベースから使用状況統計と系統情報を読み取る権限を付与します:

```sql
GRANT IMPORTED PRIVILEGES ON DATABASE SNOWFLAKE TO ROLE dbt_metadata_role;
```

## 重要な考慮事項

以下は、サードパーティシステムからのメタデータの一貫性、信頼性、およびスケーラビリティを確保するために設計された、外部メタデータ取り込みに関するベストプラクティスです。

- <Constant name="explorer" /> は、dbt と Snowflake 間の共有リソースを統合します。たとえば、dbt モデルを表す Snowflake テーブルがある場合、これらは <Constant name="explorer" /> 内で単一のリソースとして表されます。適切に統合されるためには、[本番環境](/docs/deploy/deploy-environments#set-as-production-environment) と外部メタデータ取り込み認証情報の両方で同じ接続を使用する必要があります。
- 重複を避ける: 可能であれば、プラットフォームごとに 1 つのメタデータ接続を使用します (たとえば、Snowflake 用に 1 つ、BigQuery 用に 1 つ)。
    - 同じウェアハウスを指す複数の接続があると、メタデータが重複する可能性があります。
- dbt 環境との連携：アセットの系統とメタデータを統合するには、dbt 環境と外部メタデータ取り込みの両方で同じウェアハウス接続が使用されるようにします。
- フィルターを使用して、取り込みを関連するアセットに限定します。
    - 例：本番環境スキーマのみに制限する、または一時スキーマを無視する。

外部メタデータ取り込みは、1 日に 1 回、UTC 午後 5 時に実行されることに注意してください。




