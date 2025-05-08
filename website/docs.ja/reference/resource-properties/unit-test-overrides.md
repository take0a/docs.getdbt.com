---
title: "ユニットテストのオーバーライド"
sidebar_label: "Overrides"
---

単体テストを構成するときに、特定の単体テストの [マクロ](/docs/build/jinja-macros#macros)、[プロジェクト変数](/docs/build/project-variables)、または [環境変数](/docs/build/environment-variables) の出力をオーバーライドできます。

```yml

 - name: test_my_model_overrides
    model: my_model
    given:
      - input: ref('my_model_a')
        rows:
          - {id: 1, a: 1}
      - input: ref('my_model_b')
        rows:
          - {id: 1, b: 2}
          - {id: 2, b: 2}
    overrides:
      macros:
        type_numeric: override
        invocation_id: 123
      vars:
        my_test: var_override
      env_vars:
        MY_TEST: env_var_override
    expect:
      rows:
        - {macro_call: override, var_call: var_override, env_var_call: env_var_override, invocation_id: 123}

```

## マクロ

ユニットテスト定義内のマクロの出力をオーバーライドできます。

ユニットテスト対象のモデルで以下のマクロを使用している場合は、オーバーライドする必要があります。
  - [`is_incremental`](/docs/build/incremental-models#understand-the-is_incremental-macro): 増分モデルのユニットテストを行う場合は、`is_incremental` を明示的に `true` または `false` に設定する必要があります。増分モデルのユニットテストに関する詳細なドキュメントは、[こちら](/docs/build/unit-tests#unit-testing-incremental-models) をご覧ください。

  ```yml

  unit_tests:
    - name: my_unit_test
      model: my_incremental_model
      overrides:
        macros:
          # unit test this model in "full refresh" mode
          is_incremental: false 
      ...

  ```

  - [`dbt_utils.star`](/blog/star-sql-love-letter): `star` マクロを使用するモデルのユニットテストを行う場合、`star` を明示的に列のリストに設定する必要があります。これは、`star` が `from` 引数として [relation](/reference/dbt-classes#relation) のみを受け入れるためです。ユニットテストのモック入力データはモデルのSQLに直接挿入され、`ref('')` または `source('')` 関数を置き換えます。そのため、オーバーライドしない限り `star` マクロは失敗します。

  ```yml

  unit_tests:
    - name: my_other_unit_test
      model: my_model_that_uses_star
      overrides:
        macros:
          # explicity set star to relevant list of columns
          dbt_utils.star: col_a,col_b,col_c 
      ...

  ``` 
