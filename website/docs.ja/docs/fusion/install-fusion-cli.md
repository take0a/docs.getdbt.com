---
title: "CLIからFusionをインストールする"
description: "Install the Fusion engine locally from the command line interface (CLI) to take data transformation to the next level."
keywords: ["dbt Fusion engine", "Fusion", "Install Fusion", "Update Fusion", "Fusion updates" ]
id: install-fusion-cli
---

# CLIからFusionをインストールする <Lifecycle status="preview" />

Fusion は、公式 CDN からコマンド ライン経由でインストールできます。

- **macOS/Linux:** Using `curl`
<!--- **Windows:** Using `irm` -->

## macOS & Linux インストール

ターミナルで次のコマンドを実行します。

```shell
curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update
```

インストール後すぐに `dbtf` を使用するには、新しい `$PATH` が認識されるようにシェルをリロードします。

```shell
exec $SHELL
```

または、ターミナルウィンドウを閉じて再度開きます。これにより、更新された環境設定が新しいセッションに読み込まれます。

### Windows installation (PowerShell)

PowerShell で次のコマンドを実行します。

```powershell
irm https://public.cdn.getdbt.com/fs/install/install.ps1 | iex
```

インストール後すぐに `dbtf` を使用するには、新しい `Path` が認識されるようにシェルをリロードします。

```powershell
Start-Process powershell
```

または、PowerShell を閉じて再度開きます。これにより、更新された環境設定が新しいセッションに読み込まれます。

## インストールを確認する

インストール後、新しいコマンドラインウィンドウを開き、バージョンを確認してFusionが正しくインストールされていることを確認してください。これらのコマンドは`dbt`を使用して実行できます。また、マシンに別のdbt CLIがインストールされている場合は、Fusionの明確なエイリアスとして`dbtf`を使用することもできます。

```bash
dbtf --version
```

- **macOS** & **Linux**: $HOME/.local/bin/dbt
- **Windows:** `C:\Users\<YourUsername>\.local\bin\dbt.exe`

この場所は、`dbtf` コマンドを簡単に実行できるようにパスに自動的に追加されますが、シェルをリロードする必要があります。

## Fusion を更新

以下のコマンドを実行すると、Fusion とアダプタコードが最新バージョンに更新されます。

```shell
dbtf system update
```

## Fusion のアンインストール

このコマンドはシステムから Fusion バイナリをアンインストールしますが、エイリアスはインストールされた場所（例: `~/.zshrc`）に残ります。

```shell
dbtf system uninstall
```

## アダプタのインストール

Fusion のインストールには、[Fusion の要件](/docs/fusion/supported-features#requirements) に記載されているアダプタが自動的に含まれます。その他のアダプタは後日提供開始予定です。

## トラブルシューティング

よくある問題と解決策:

- **dbt コマンドが見つかりません:** インストール場所が `$PATH` に正しく追加されていることを確認してください。
- **バージョンの競合:** Fusion と競合する可能性のある既存の <Constant name="core" /> または dbt CLI バージョンがインストール (またはアクティブ) されていないことを確認してください。
- **インストール権限:** ユーザーにソフトウェアをローカルにインストールするための適切な権限があることを確認してください。

## よくある質問

- 以前の dbt インストールに戻すことはできますか？

    はい。既存のワークフローに影響を与えずに Fusion をテストしたい場合は、インストールを別の環境または仮想マシンで分離または管理することを検討してください。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />
