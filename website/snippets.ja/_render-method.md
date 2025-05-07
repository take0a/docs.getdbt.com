#### render メソッド

`.render()` メソッドは通常、実行時に Jinja 式（`{{ source(...) }}` など）を解決または評価するために使用されます。

`--empty フラグ` を使用すると、dbt は最適化のために `ref()` または `source()` の処理を​​スキップすることがあります。コンパイルエラーを回避し、特定のリレーション（`ref()` または `source()`）を処理するように dbt に明示的に指示するには、モデルファイルで `.render()` メソッドを使用します。例:


<File name='models.sql'>

```Jinja
{{ config(
    pre_hook = [
        "alter external table {{ source('sys', 'customers').render() }} refresh"
    ]
) }}

select ...
```

</File>
