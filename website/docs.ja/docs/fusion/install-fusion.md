---
title: "Install Fusion"
description: "Install the Fusion engine locally to take data transformation to the next level."
id: install-fusion
---

# Fusionのインストールについて <Lifecycle status="beta" />

import FusionBeta from '/snippets.ja/_fusion-beta-callout.md';
import FusionDWH from '/snippets/_fusion-dwh.md';
import FusionA from '/snippets/_fusion-auth.md';

<FusionBeta />

このガイドでは、重要な前提条件、段階的なインストール手順、一般的な問題のトラブルシューティング、構成ガイダンスなど、Fusion をローカルにインストールする手順について説明します。

## 前提条件

Fusion をインストールする前に、以下の点をご確認ください。

- ローカルマシンにソフトウェアをインストールするための管理者権限を持っていること。
- コマンドラインインターフェース（macOS/Linux の場合はターミナル、Windows の場合は PowerShell）に精通していること。
- サポートされているアダプターを使用していること。今後、さらに多くのアダプターのサポートが追加される予定です。
  <FusionDWH /> 
- サポートされている認証方法を使用しています:
  <FusionA /> 

## Install Fusion

Fusion は、公式 CDN からコマンド ライン経由でインストールできます:

- **macOS/Linux:** Using `curl`
- **Windows:** Using `irm`

### macOSおよびLinuxへのインストール

ターミナルで次のコマンドを実行します:

```shell
curl -fsSL https://public.cdn.getdbt.com/fs/install/install.sh | sh -s -- --update
```

インストール後すぐに `dbtf` を使用するには、新しい `$PATH` が認識されるようにシェルをリロードします。

```shell
exec $SHELL
```

または、ターミナルウィンドウを閉じて再度開きます。これにより、更新された環境設定が新しいセッションに読み込まれます。

### Windows インストール (PowerShell)

PowerShell で次のコマンドを実行します:

```powershell
irm https://public.cdn.getdbt.com/fs/install/install.ps1 | iex
```

インストール後すぐに `dbtf` を使用するには、新しい `Path` が認識されるようにシェルをリロードします:

```powershell
Start-Process powershell
```

または、PowerShell を閉じて再度開きます。これにより、更新された環境設定が新しいセッションに読み込まれます。

### インストールの確認

インストール後、新しいコマンドラインウィンドウを開き、バージョンを確認して Fusion が正しくインストールされていることを確認してください。これらのコマンドは `dbt` を使用して実行できます。また、マシンに別の dbt CLI がインストールされている場合は、Fusion の明確なエイリアスとして `dbtf` を使用することもできます。

```bash
dbtf --version
```

Fusion は次の場所にインストールされます:

- **macOS & Linux:** `$HOME/.local/bin/dbt`
- **Windows:** `C:\Users\<YourUsername>\.local\bin\dbt.exe`

この場所は、`dbtf` コマンドを簡単に実行できるようにパスに自動的に追加されますが、シェルをリロードする必要があります。

### Fusion を更新

次のコマンドを実行すると、Fusion とアダプタコードが最新バージョンに更新されます:

```shell
dbtf system update
```

### アンインストール

このコマンドは、システムから Fusion バイナリをアンインストールします（ただし、エイリアスはインストールされた場所に残ります（例: `~/.zshrc`）。

```shell
dbtf system uninstall
```

### アダプタのインストール

Fusion のインストールには、Snowflake アダプタが自動的に含まれます。その他のアダプタは後日提供開始予定です。

## トラブルシューティング

よくある問題と解決策:

- **dbt コマンドが見つかりません:** インストール場所が `$PATH` に正しく追加されていることを確認してください。
- **バージョンの競合:** Fusion と競合する可能性のある既存の dbt Core または dbt Cloud CLI バージョンがインストール (またはアクティブ) されていないことを確認してください。
- **インストール権限:** ユーザーにソフトウェアをローカルにインストールするための適切な権限があることを確認してください。

## Frequently asked questions

- 以前の dbt インストールに戻すことはできますか？

    はい。既存のワークフローに影響を与えずに Fusion をテストしたい場合は、インストールを別の環境または仮想マシンで分離または管理することを検討してください。

import AboutFusion from '/snippets.ja/_about-fusion.md';

<AboutFusion />
