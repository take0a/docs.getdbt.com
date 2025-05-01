---
title: "分析"
description: "分析に使用するコンパイル済みコードを作成するには、dbt で SQL ファイルを構成します。"
id: "analyses"
pagination_next: null
---

## 概要

dbt の「モデル」という概念により、データチームはデータ変換におけるバージョン管理と共同作業を容易に行うことができます。
しかしながら、特定の SQL 文が dbt モデルの型に完全には当てはまらない場合があります。
こうした「分析的な」SQL ファイルは、dbt の「分析」機能を使用して、dbt プロジェクト内でバージョン管理できます。

dbt プロジェクトの `analyses/` ディレクトリにある `.sql` ファイルはコンパイルされますが、実行されません。
つまり、アナリストは `{{ ref(...) }}` などの dbt 機能を使用して、環境に依存しない方法でモデルを選択できます。

実際には、分析ファイルは次のようになります（[オープンソースの Quickbooks モデル](https://github.com/dbt-labs/quickbooks) を使用）。

<File name='analyses/running_total_by_account.sql'>

```sql
-- analyses/running_total_by_account.sql

with journal_entries as (

  select *
  from {{ ref('quickbooks_adjusted_journal_entries') }}

), accounts as (

  select *
  from {{ ref('quickbooks_accounts_transformed') }}

)

select
  txn_date,
  account_id,
  adjusted_amount,
  description,
  account_name,
  sum(adjusted_amount) over (partition by account_id order by id rows unbounded preceding)
from journal_entries
order by account_id, id
```

</File>

この分析を実行可能な SQL にコンパイルするには、次を実行します:
```
dbt compile
```

次に、`target/compiled/{プロジェクト名}/analyses/running_total_by_account.sql` でコンパイル済みのSQLファイルを探します。
このSQLは、例えばデータ視覚化ツールに貼り付けることができます。
これは `model` ではなく `analysis` であるため、`running_total_by_account` リレーションはデータベースに実体化されないことに注意してください。
