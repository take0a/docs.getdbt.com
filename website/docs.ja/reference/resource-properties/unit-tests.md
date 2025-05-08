---
title: "unit test プロパティについて"
sidebar_label: "Unit tests"
resource_types: [models]
datatype: test
---

<VersionCallout version="1.8" />

ユニットテストは、本番環境で完全なモデルを実装する前に、少数の静的入力セットを使用して SQL モデリングロジックを検証します。テスト駆動開発アプローチをサポートし、開発者の効率とコードの信頼性の両方を向上させます。

ユニットテストのみを実行するには、次のコマンドを使用します。
`dbt test --select test_type:unit`

## 始める前に

- 現在、SQL モデルのユニットテストのみをサポートしています。
- 現在、ユニットテストの追加は、_現在の_プロジェクト内のモデルに対してのみサポートしています。
- モデルに複数のバージョンがある場合、デフォルトではユニットテストはモデルの*すべての*バージョンで実行されます。詳細については、[バージョン管理されたモデルのユニットテスト](/reference/resource-properties/unit-testing-versions)を参照してください。
- ユニットテストは、`models/` ディレクトリ内の YML ファイルで定義する必要があります。
- 一時モデルに依存するモデルのユニットテストを行う場合は、その入力に `format: sql` を使用する必要があります。

<file name='dbt_project.yml'>

```yml

unit_tests:
  - name: <test-name> # this is the unique name of the test
    model: <model-name> 
      versions: #optional
        include: <list-of-versions-to-include> #optional
        exclude: <list-of-versions-to-exclude> #optional
    config: 
      meta: {dictionary}
      tags: <string> | [<string>]
      enabled: {boolean} # optional. v1.9 or higher. If not configured, defaults to `true`
    given:
      - input: <ref_or_source_call> # optional for seeds
        format: dict | csv | sql
        # either define rows inline or name of fixture
        rows: {dictionary} | <string>
        fixture: <fixture-name> # sql or csv 
      - input: ... # declare additional inputs
    expect:
      format: dict | csv | sql
      # either define rows inline of rows or name of fixture
      rows: {dictionary} | <string>
      fixture: <fixture-name> # sql or csv 
    overrides: # optional: configuration for the dbt execution environment
      macros:
        is_incremental: true | false
        dbt_utils.current_timestamp: <string>
        # ... any other jinja function from https://docs.getdbt.com/reference/dbt-jinja-functions
        # ... any other context property
      vars: {dictionary}
      env_vars: {dictionary}
  - name: <test-name> ... # declare additional unit tests

  ```

</file>

## 例

```yml

unit_tests:
  - name: test_is_valid_email_address # this is the unique name of the test
    model: dim_customers # name of the model I'm unit testing
    given: # the mock data for your inputs
      - input: ref('stg_customers')
        rows:
         - {email: cool@example.com,     email_top_level_domain: example.com}
         - {email: cool@unknown.com,     email_top_level_domain: unknown.com}
         - {email: badgmail.com,         email_top_level_domain: gmail.com}
         - {email: missingdot@gmailcom,  email_top_level_domain: gmail.com}
      - input: ref('top_level_email_domains')
        rows:
         - {tld: example.com}
         - {tld: gmail.com}
    expect: # the expected output given the inputs above
      rows:
        - {email: cool@example.com,    is_valid_email_address: true}
        - {email: cool@unknown.com,    is_valid_email_address: false}
        - {email: badgmail.com,        is_valid_email_address: false}
        - {email: missingdot@gmailcom, is_valid_email_address: false}

```

```yml

unit_tests:
  - name: test_is_valid_email_address # this is the unique name of the test
    model: dim_customers # name of the model I'm unit testing
    given: # the mock data for your inputs
      - input: ref('stg_customers')
        rows:
         - {email: cool@example.com,     email_top_level_domain: example.com}
         - {email: cool@unknown.com,     email_top_level_domain: unknown.com}
         - {email: badgmail.com,         email_top_level_domain: gmail.com}
         - {email: missingdot@gmailcom,  email_top_level_domain: gmail.com}
      - input: ref('top_level_email_domains')
        format: csv
        rows: |
          tld
          example.com
          gmail.com
    expect: # the expected output given the inputs above
      format: csv
      fixture: valid_email_address_fixture_output

```

```yml

unit_tests:
  - name: test_is_valid_email_address # this is the unique name of the test
    model: dim_customers # name of the model I'm unit testing
    given: # the mock data for your inputs
      - input: ref('stg_customers')
        rows:
         - {email: cool@example.com,     email_top_level_domain: example.com}
         - {email: cool@unknown.com,     email_top_level_domain: unknown.com}
         - {email: badgmail.com,         email_top_level_domain: gmail.com}
         - {email: missingdot@gmailcom,  email_top_level_domain: gmail.com}
      - input: ref('top_level_email_domains')
        format: sql
        rows: |
          select 'example.com' as tld union all
          select 'gmail.com' as tld
    expect: # the expected output given the inputs above
      format: sql
      fixture: valid_email_address_fixture_output

```
