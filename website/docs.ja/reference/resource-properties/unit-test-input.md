---
title: "ユニットテストの入力"
sidebar_label: "Input"
---

ユニットテストで入力を使用して、テストの特定のモデルまたはソースを参照します。

- `input:` には、`ref` または `source` 呼び出しを表す文字列を使用します。
  - `ref('my_model')` または `ref('my_model', v='2')` または `ref('dougs_project', 'users')`
  - `source('source_schema', 'source_name')`
- オプションで seed に使用します。
  - seed に入力を指定しない場合は、seed が入力として使用されます。
  - seed に入力を指定した場合は、代わりにその入力が使用されます。
- rows に空のリスト `rows: []` を設定することで、「空」の入力を使用します。

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
...

```