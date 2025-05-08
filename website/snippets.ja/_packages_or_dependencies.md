
## ユースケース

以下の設定は、すべての dbt プロジェクトで機能します。

- [任意のパッケージ依存関係](/docs/collaborate/govern/project-dependencies#when-to-use-project-dependencies) を `packages.yml` に追加します。
- [任意のプロジェクト依存関係](/docs/collaborate/govern/project-dependencies#when-to-use-package-dependencies) を `dependencies.yml` に追加します。

ただし、両方を 1 つの `dependencies.yml` ファイルに統合できる場合があります。詳細については、次のセクションをご覧ください。

#### packages.yml と dependency.yml について
`dependencies.yml` ファイルには、「パッケージ」依存関係と「プロジェクト」依存関係の両方の種類の依存関係を含めることができます。
- [パッケージ依存関係](/docs/build/packages#how-do-i-add-a-package-to-my-project) を使用すると、ライブラリのように、他の dbt プロジェクトのソースコードを自分のプロジェクトに追加できます。
- プロジェクト依存関係は、他のユーザーが dbt で作成したソースコードを基にビルドを行う別の方法を提供します。

dbt プロジェクトでパッケージ仕様内で Jinja を使用する必要がない場合は、既存の `packages.yml` の名前を `dependencies.yml` に変更するだけで済みます。ただし、プロジェクトのパッケージ仕様で Jinja を使用している場合、特にプライベート Git パッケージ仕様に環境変数や [Git トークンメソッド](/docs/build/packages#git-token-method) を追加するようなシナリオでは、引き続き `packages.yml` ファイル名を使用する必要があります。

以下のトグルを使用して違いを理解し、`dependencies.yml` と `packages.yml`（あるいは両方）のどちらを使用するかを判断してください。詳細については、[FAQ](#faqs) を参照してください。

<Expandable alt_header="プロジェクトの依存関係を使用する場合" >

プロジェクト依存関係は、[dbt Mesh](/best-practices/how-we-mesh/mesh-1-intro) および [プロジェクト間参照](/docs/collaborate/govern/project-dependencies#how-to-write-cross-project-ref) ワークフロー向けに設計されています。

- 異なる dbt プロジェクト間、特に dbt Mesh セットアップでプロジェクト間参照を設定する必要がある場合は、`dependencies.yml` を使用します。
- プロジェクトの依存関係にプロジェクトと非プライベート dbt パッケージの両方を含める場合は、`dependencies.yml` を使用します。
- プライベートパッケージは、Jinja レンダリングや条件付き構成を意図的にサポートしていないため、`dependencies.yml` ではサポートされていません。これは、静的で予測可能な構成を維持し、dbt Cloud などの他のサービスとの互換性を確保するためです。
- [プロジェクト間参照](/docs/collaborate/govern/project-dependencies#how-to-write-cross-project-ref)と[dbt Hubパッケージ](https://hub.getdbt.com/)の両方を使用している場合は、整理と保守性のために`dependencies.yml`を使用してください。これにより、依存関係を管理するために複数のYAMLファイルを作成する必要性が軽減されます。

</Expandable>

<Expandable alt_header="パッケージ依存関係を使用する場合" >

パッケージ依存関係を使用すると、ライブラリのように、他の dbt プロジェクトのソースコードを自分のプロジェクトに追加できます。

- [dbt Hub](https://hub.getdbt.com/) などのパッケージのみを使用する場合は、`packages.yml` をそのまま使用してください。
- dbt プロジェクトなどの dbt パッケージをルートまたは親 dbt プロジェクトにダウンロードする場合は、`packages.yml` を使用します。ただし、これは dbt Mesh ワークフローには影響しないことに注意してください。
- プロジェクトの依存関係にパッケージ（プライベートパッケージを含む）を含めるには、`packages.yml` を使用します。参照する必要があるプライベートパッケージがある場合は、`packages.yml` を使用することをお勧めします。
- `packages.yml` は、歴史的な理由から Jinja レンダリングをサポートしており、動的な構成が可能です。これは、[Git トークンメソッド](/docs/build/packages#git-token-method) のような環境変数から値をパッケージ仕様に挿入する必要がある場合に便利です。

現在、dbt でプライベート Git リポジトリを使用するには、Jinja を使用して Git トークンを埋め込む回避策を使用する必要があります。これは、ユーザーの作成や Git トークンの共有といった追加の手順が必要となるため、理想的ではありません。近日中に、Jinja に埋め込まれたシークレット環境変数を必要としない、よりシンプルな方法を導入する予定です。そのため、`dependencies.yml` は Jinja をサポートしていません。

</Expandable>
