---
title: "プログラムによる呼び出し"
---

v1.5 では、dbt-core にプログラムによる呼び出しのサポートが追加されました。これは、既存の dbt Core CLI を Python エントリポイント経由で公開し、Python スクリプトまたはアプリケーション内からトップレベルのコマンドを呼び出せるようにすることを目的としています。

エントリポイントは `dbtRunner` クラスであり、これを使用することで CLI と同じコマンドを `invoke` できます。

```python
from dbt.cli.main import dbtRunner, dbtRunnerResult

# initialize
dbt = dbtRunner()

# create CLI args as a list of strings
cli_args = ["run", "--select", "tag:my_tag"]

# run the command
res: dbtRunnerResult = dbt.invoke(cli_args)

# inspect the results
for r in res.result:
    print(f"{r.node.name}: {r.status}")
```

## 並列実行はサポートされていません

[`dbt-core`](https://pypi.org/project/dbt-core/) は、同一プロセス内での複数の呼び出しに対する [安全な並列実行](/reference/dbt-commands#parallel-execution) をサポートしていません。つまり、複数の dbt コマンドを同時に実行することは安全ではありません。これは公式には推奨されておらず、サブプロセスを処理するにはラッピングプロセスが必要です。その理由は以下のとおりです。

- コマンドを同時に実行すると、データプラットフォームと予期しない相互作用が生じる可能性があります。たとえば、同じモデルに対して `dbt run` と `dbt build` を同時に実行すると、予期しない結果が生じる可能性があります。
- 各 `dbt-core` コマンドは、グローバル Python 変数と相互作用します。安全な操作を確保するには、コマンドを別々のプロセスで実行する必要があります。これは、プロセスの生成などの方法や、Celery などのツールを使用することで実現できます。

[安全な並列実行](/reference/dbt-commands#available-commands)を実行するには、[dbt Cloud CLI](/docs/cloud/cloud-cli-installation) または [dbt Cloud IDE](/docs/cloud/dbt-cloud-ide/develop-in-the-cloud) を使用できます。どちらも、同時実行性 (複数のプロセス) を管理するための追加作業を自動的に実行します。

## `dbtRunnerResult`

各コマンドは `dbtRunnerResult` オブジェクトを返します。このオブジェクトには次の 3 つの属性があります。
- `success` (bool): コマンドが成功したかどうか。
- `result`: コマンドが完了した場合 (正常に完了した場合、またはエラーが処理された場合)、その結果。戻り値の型はコマンドによって異なります。
- `exception`: dbt 呼び出しで未処理のエラーが発生し、完了しなかった場合、発生した例外。

[CLI 終了コード](/reference/exit-codes) とプログラムによる呼び出しで返される `dbtRunnerResult` は 1:1 に対応しています。

| Scenario                                                                                    | CLI Exit Code | `success` | `result`         | `exception` |
|---------------------------------------------------------------------------------------------|--------------:|-----------|-------------------|-------------|
| 呼び出しはエラーなしで完了しました  | 0             | `True`      | varies by command | `None`        |
| 少なくとも 1 つの処理済みエラー (例: テスト失敗、モデル ビルド エラー) を伴って呼び出しが完了しました | 1             | `False`     | varies by command | `None`        |
| 処理されないエラーです。呼び出しが完了しなかったため、結果が返されません。 | 2             | `False`     | `None`              | Exception   |

## コミットメントと注意事項

dbt Core v1.5以降、dbt-coreのCLIと同等の機能を持つPythonエントリポイントを提供することに継続的に取り組んでいます。この目標を達成するために使用する基盤となる実装は、変更する権利を留保します。現在の実装は、短期および中期的には実際のユースケースに対応していくと期待しており、同時に、最終的に現在の実装に代わる安定した長期的なインターフェースの開発に取り組んでいます。

特に、各コマンドによって返される`dbtRunnerResult.result`内のオブジェクトは完全にはコントラクト化されていないため、変更される可能性があります。返されるオブジェクトの一部は、[dbtアーティファクト](/reference/artifacts/dbt-artifacts)の内容と一部重複しているため、部分的にドキュメント化されています。Pythonオブジェクトであるため、シリアル化されたJSONアーティファクトで利用できるものよりも多くのフィールドとメソッドが含まれています。これらの追加フィールドとメソッドは**内部仕様であり、dbt-coreの将来のバージョンで変更される可能性があります。**

## 高度な使用パターン

:::caution
これらのパターンの構文とサポートは、`dbt-core` の将来のバージョンで変更される可能性があります。
:::

`dbtRunner` の目標は、プログラム環境内で CLI ワークフローと同等の機能を提供することです。CLI の可能性を拡張する高度な使用パターンもいくつかあります。

### オブジェクトの再利用

ディスクからファイルを読み込んでオブジェクトを再作成する必要がないように、事前に構築されたオブジェクトを `dbtRunner` に渡します。現在サポートされているオブジェクトは `Manifest`（プロジェクトコンテンツ）のみです。

```python
from dbt.cli.main import dbtRunner, dbtRunnerResult
from dbt.contracts.graph.manifest import Manifest

# use 'parse' command to load a Manifest
res: dbtRunnerResult = dbtRunner().invoke(["parse"])
manifest: Manifest = res.result

# introspect manifest
# e.g. assert every public model has a description
for node in manifest.nodes.values():
    if node.resource_type == "model" and node.access == "public":
        assert node.description != "", f"{node.name} is missing a description"

# reuse this manifest in subsequent commands to skip parsing
dbt = dbtRunner(manifest=manifest)
cli_args = ["run", "--select", "tag:my_tag"]
res = dbt.invoke(cli_args)
```

### コールバックの登録

構造化イベントにアクセスし、カスタムログを有効にするには、dbt の `EventManager` に `callbacks` を登録します。コールバックの現在の動作では、後続のステップがブロックされます。この機能は将来のバージョンでは保証されません。

<VersionBlock firstVersion="1.8">

```python
from dbt.cli.main import dbtRunner
from dbt_common.events.base_types import EventMsg

def print_version_callback(event: EventMsg):
    if event.info.name == "MainReportVersion":
        print(f"We are thrilled to be running dbt{event.data.version}")

dbt = dbtRunner(callbacks=[print_version_callback])
dbt.invoke(["list"])
```

</VersionBlock>

<VersionBlock lastVersion="1.7">

```python
from dbt.cli.main import dbtRunner
from dbt.events.base_types import EventMsg

def print_version_callback(event: EventMsg):
    if event.info.name == "MainReportVersion":
        print(f"We are thrilled to be running dbt{event.data.version}")

dbt = dbtRunner(callbacks=[print_version_callback])
dbt.invoke(["list"])
```

</VersionBlock>

### パラメータのオーバーライド

パラメータは、CLI 形式の文字列リストではなく、キーワード引数として渡してください。現時点では、dbt は入力値の検証や型変換を行いません。サブコマンドは、最初の位置引数としてリスト形式で指定する必要があります。

```python
from dbt.cli.main import dbtRunner
dbt = dbtRunner()

# these are equivalent
dbt.invoke(["--fail-fast", "run", "--select", "tag:my_tag"])
dbt.invoke(["run"], select=["tag:my_tag"], fail_fast=True)
```
