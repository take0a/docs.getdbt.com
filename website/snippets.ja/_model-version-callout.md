
:::info モデルバージョン、dbt_project.yml バージョン、および .yml バージョン

[モデルバージョン](/docs/collaborate/govern/model-versions) は、[dbt_project.yml バージョン](/reference/project-configs/version#dbt_projectyml-versions) や [.yml プロパティファイルバージョン](/reference/project-configs/version#yml-property-file-versions) とは異なることに注意してください。

モデルバージョンは、モデルの変更や更新を経時的に追跡できるようにすることで、ガバナンスとデータモデル管理を向上させる _機能_ です。dbt_project.yml バージョンは、dbt プロジェクトと特定のバージョンの dbt との互換性を示します。.yml プロパティファイル内のバージョン番号は、dbt がそれらの YAML ファイルをどのように解析するかを示します。後者 2 つは、dbt v1.5 以降では完全にオプションです。

:::
