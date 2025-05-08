---
title: "dbt parse コマンドについて"
sidebar_label: "parse"
description: "dbt の parse コマンドを使用して dbt プロジェクトを解析し、詳細なタイミング情報を書き込む方法については、このガイドをお読みください。"
id: "parse"
---

`dbt parse` コマンドは、dbt プロジェクトの内容を解析および検証します。プロジェクトに Jinja または YAML 構文エラーが含まれている場合、コマンドは失敗します。

また、詳細なタイミング情報を含むアーティファクトも生成されます。これは、大規模プロジェクトの解析時間を把握するのに役立ちます。詳細については、[プロジェクトの解析](/reference/parsing) を参照してください。

v1.5 以降、`dbt parse` は [マニフェスト](/reference/artifacts/manifest-json) を書き込んだり返したりするようになりました。これにより、プロジェクト内のすべてのリソースに対する dbt の理解状況を把握できます。`dbt parse` はウェアハウスに接続しないため、[このマニフェストにはコンパイル済みコードは含まれません](/faqs/Warehouse/db-connection-dbt-compile)。

デフォルトでは、dbt Cloud IDE は「部分的な」解析を試みます。つまり、前回の解析以降の変更（プロジェクトに変更を加えた際に追加された部分または更新された部分）のみをチェックします。dbt Cloud IDE は作業を保存するたびにバックグラウンドで自動的に解析を行うため、`dbt parse` を手動で実行すると、最近の変更のみを確認するため、処理が速くなる可能性があります。

オプションとして、`--no-partial-parse` フラグを使用して、dbt にプロジェクト全体を最初からチェックするように指示することもできます。これにより、dbt は最近の変更だけでなく、プロジェクト全体を再解析します。

```
$ dbt parse
13:02:52  Running with dbt=1.5.0
13:02:53  Performance info: target/perf_info.json
```

<File name='target/perf_info.json'>

```json
{
    "path_count": 7,
    "is_partial_parse_enabled": false,
    "parse_project_elapsed": 0.20151838900000008,
    "patch_sources_elapsed": 0.00039490800000008264,
    "process_manifest_elapsed": 0.029363873999999957,
    "load_all_elapsed": 0.240095269,
    "projects": [
        {
            "project_name": "my_project",
            "elapsed": 0.07518750299999999,
            "parsers": [
                {
                    "parser": "model",
                    "elapsed": 0.04545303199999995,
                    "path_count": 1
                },
                {
                    "parser": "operation",
                    "elapsed": 0.0006415469999998535,
                    "path_count": 1
                },
                {
                    "parser": "seed",
                    "elapsed": 0.026538173000000054,
                    "path_count": 2
                }
            ],
            "path_count": 4
        },
        {
            "project_name": "dbt_postgres",
            "elapsed": 0.0016448299999998195,
            "parsers": [
                {
                    "parser": "operation",
                    "elapsed": 0.00021672399999994596,
                    "path_count": 1
                }
            ],
            "path_count": 1
        },
        {
            "project_name": "dbt",
            "elapsed": 0.006580432000000025,
            "parsers": [
                {
                    "parser": "operation",
                    "elapsed": 0.0002488560000000195,
                    "path_count": 1
                },
                {
                    "parser": "docs",
                    "elapsed": 0.002500640000000054,
                    "path_count": 1
                }
            ],
            "path_count": 2
        }
    ]
}
```

</File>
