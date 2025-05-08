---
resource_types: [seeds]
datatype: <string>
default_value: ","
---

<VersionCallout version="1.7" />

## 定義

このオプションの seed 設定を使用すると、[ seed ](/docs/build/seeds) 内の値の区切り方を、1文字の文字列でカスタマイズできます。

* 区切り文字を指定しない場合は、デフォルトでカンマが使用されます。
*  seed ファイルで「|」や「;」などの別の区切り文字を使用する場合は、「delimiter」設定値を明示的に設定してください。
  
## 使用方法

`dbt_project.yml` ファイルで区切り文字を指定して、すべての seed 値のグローバル区切り文字をカスタマイズします。

<File name='dbt_project.yml'>

```yml
seeds:
  <project_name>:
     +delimiter: "|" # default project delimiter for seeds will be "|"
    <seed_subdirectory>:
      +delimiter: "," # delimiter for seeds in seed_subdirectory will be ","
```

</File>


または、カスタム区切り文字を使用して、特定の seed の値を上書きします:

<File name='seeds/properties.yml'>

```yml
version: 2

seeds:
  - name: <seed_name>
    config: 
      delimiter: "|"
```

</File>

## Examples
For a project with:

* `name: jaffle_shop` in the `dbt_project.yml` file
* `seed-paths: ["seeds"]` in the `dbt_project.yml` file

### カスタム区切り文字を使用してグローバル値を上書き

カンマを使用する seed 「seed_a」を除き、すべての seed に対してデフォルトの動作を設定できます:

<File name='dbt_project.yml'>

```yml
seeds:
  jaffle_shop: 
    +delimiter: "|" # default delimiter for seeds in jaffle_shop project will be "|"
    seed_a:
      +delimiter: "," # delimiter for seed_a will be ","
```

</File>

対応する seed  ファイルは次のようにフォーマットされます:

<File name='seeds/my_seed.csv'>

```text
col_a|col_b|col_c
1|2|3
4|5|6
...
```

</File>

<File name='seeds/seed_a.csv'>

```text
name,id
luna,1
doug,2
...
```

</File>

あるいは、1つの seed に対してカスタム動作を設定することもできます。`country_codes` では「;」区切り文字を使用します:

<File name='seeds/properties.yml'>

```yml
version: 2

seeds:
  - name: country_codes
    config:
      delimiter: ";"
```

</File>

`country_codes`  seed  ファイルは次のようにフォーマットされます:

<File name='seeds/country_codes.csv'>

```text
country_code;country_name
US;United States
CA;Canada
GB;United Kingdom
...
```

</File>
