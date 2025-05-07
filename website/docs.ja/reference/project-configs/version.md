---
datatype: version
required: True
keyword: project version, project versioning, dbt project versioning
---

import VersionsCallout from '/snippets.ja/_model-version-callout.md';

<VersionsCallout />


dbt プロジェクトには 2 つの異なるタイプの `version` タグがあります。このフィールドは、その場所によって意味が異なります。

## `dbt_project.yml` のバージョン

`dbt_project` ファイル内のバージョンタグは、dbt プロジェクトのバージョンを表します。

dbt バージョン 1.5 以降、`dbt_project.yml` 内の `version` は *オプションパラメータ* です。バージョンを指定する場合は、`1.0.0` などの [セマンティックバージョン](https://semver.org/) 形式にする必要があります。指定されていない場合のデフォルト値は `None` です。dbt バージョン 1.4 以前のバージョンをご利用の場合、このタグは必須ですが、現時点では dbt では意味のある意味で使用されていません。

Core バージョンの詳細については、[dbt Core バージョンについて](/docs/dbt-versions/core) を参照してください。

<File name='dbt_project.yml'>

```yml
version: version
```

</File>

## `.yml` プロパティファイルのバージョン

`.yml` プロパティファイル内のバージョンタグは、dbt がプロパティファイルを処理する方法を指示する制御タグを提供します。

バージョン 1.5 以降、dbt はリソース `.yml` ファイルでこの設定を必要としなくなります。このタグが以前必須だった理由について詳しくは、[FAQ](#faqs) を参照してください。dbt バージョン 1.4 以前のユーザーの場合、このタグは必須です。

プロパティファイルの詳細については、同じページにある一般的な [ドキュメント](/reference/define-properties) を参照してください。

<Tabs
  groupId="resource-version-configs"
  defaultValue="version-specified"
  values={[
    { label: 'Resource property file with version specified', value: 'version-specified', },
    { label: 'Resource property file without version specified', value: 'no-version-specified', },
  ]
}>
<TabItem value="version-specified">

<File name='<any valid filename>.yml'>

```yml
version: 2  # Only 2 is accepted by dbt versions up to 1.4.latest.

models: 
    ...
```

</File>

</TabItem>

<TabItem value="no-version-specified">

<File name='<any valid filename>.yml'>

```yml

models: 
    ...
```

</File>

</TabItem>

</Tabs>

## FAQS

<FAQ path="Project/why-version-2" />
