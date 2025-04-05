これで、プロジェクトを初期化したときに dbt が作成したファイルを削除できます。

1. `models/example/` ディレクトリを削除します。
2. `dbt_project.yml` ファイルから `example:` キーと、その下にリストされているすべての構成を削除します。

    <File name='dbt_project.yml'>

    ```yaml
    # before
    models:
      jaffle_shop:
        +materialized: table
        example:
          +materialized: view
    ```

    </File>

    <File name='dbt_project.yml'>

    ```yaml
    # after
    models:
      jaffle_shop:
        +materialized: table
    ```

    </File>

3. 変更を保存します。

#### FAQs

<FAQ path="Models/removing-deleted-models" />
<FAQ path="Troubleshooting/unused-model-configurations" />
