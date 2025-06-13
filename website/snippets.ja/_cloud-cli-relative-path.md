<Constant name="cloud_cli" /> は現在、[`packages.yml` ファイル](/docs/build/packages) 内の相​​対パスをサポートしていません。代わりに、このシナリオで相対パスをサポートする [<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用してください。

以下は、<Constant name="cloud_cli" /> では機能しない `packages.yml` 内の [ローカルパッケージ](/docs/build/packages#local-packages) 構成の例です。

```yaml
# repository_root/my_dbt_project_in_a_subdirectory/packages.yml

packages:
  - local: ../shared_macros
```

この例では、`../shared_macros` は相対パスであり、dbt に以下を検索するよう指示します。
- `..` &mdash; 1 つ上のディレクトリ（`repository_root`）に移動します。
- `/shared_macros` &mdash; ルートディレクトリ内の `shared_macros` フォルダを見つけます。

この制限を回避するには、[<Constant name="cloud_ide" />](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用します。これは、`packages.yml` 内の相対パスを完全にサポートします。
