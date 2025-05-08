---
title: "ユニットテストでサポートされているデータ形式"
sidebar_label: "Data formats"
---

現在、dbt のユニットテスト用のモックデータは、以下の 3 つの形式をサポートしています:

- `dict` (デフォルト): インライン辞書値。
- `csv`: インライン CSV 値または CSV ファイル。
- `sql`: インライン SQL クエリまたは SQL ファイル。注: この形式では、すべての行のモックデータを提供する必要があります。

## dict

`format` が定義されていない場合、`dict` データ形式がデフォルトになります。

`dict` では、`rows` にインライン辞書が必要です。

```yml

unit_tests:
  - name: test_my_model
    model: my_model
    given:
      - input: ref('my_model_a')
        format: dict
        rows:
          - {id: 1, name: gerda}
          - {id: 2, b: michelle}    

```

## csv

`csv` 形式を使用する場合、`rows` にインライン CSV 文字列を使用できます:

```yml

unit_tests:
  - name: test_my_model
    model: my_model
    given:
      - input: ref('my_model_a')
        format: csv
        rows: |
          id,name
          1,gerda
          2,michelle

```

または、プロジェクトの `tests/fixtures` ディレクトリ (または構成された `test-paths` の場所) にある `fixture` の CSV ファイルの名前を指定することもできます:

```yml

unit_tests:
  - name: test_my_model
    model: my_model
    given:
      - input: ref('my_model_a')
        format: csv
        fixture: my_model_a_fixture

```

## sql

この形式を使用すると、以下のメリットがあります:
- ユニットテストできるデータの種類に対して柔軟性が高まります。
- 一時的なモデルに依存するモデルのユニットテストが可能になります。

ただし、`format: sql` を使用する場合は、_すべての行_ に対してモックデータを提供する必要があります。

`sql` 形式を使用する場合、`rows` に対してインライン SQL クエリを使用できます:

```yml

unit_tests:
  - name: test_my_model
    model: my_model
    given:
      - input: ref('my_model_a')
        format: sql
        rows: |
          select 1 as id, 'gerda' as name, null as loaded_at union all
          select 2 as id, 'michelle', null as loaded_at as name

```

または、プロジェクトの `tests/fixtures` ディレクトリ (または構成された `test-paths` の場所) にある SQL ファイルの名前を `fixture` に指定することもできます:

```yml

unit_tests:
  - name: test_my_model
    model: my_model
    given:
      - input: ref('my_model_a')
        format: sql
        fixture: my_model_a_fixture

```

**注意:** Jinja は、ユニット テストの SQL フィクスチャではサポートされていません。
