状態ディレクトリは、dbt v1.9 以降、または [dbt Cloud の「最新」リリーストラック](/docs/dbt-versions/cloud-release-tracks) を使用してビルドする必要があります。また、dbt_project.yml 内で `state_modified_compare_more_unrendered_values` を `true` に設定する必要があります。

状態ディレクトリが古いバージョンの dbt を使用してビルドされた場合、または `state_modified_compare_more_unrendered_values` 動作変更フラグが設定されていないか `false` に設定されている場合、`state:modified` との状態比較中に誤検知を回避するために、状態ディレクトリを再構築する必要があります。
