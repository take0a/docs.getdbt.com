---
title: "コマンドラインオプション"
id: "command-line-options"
sidebar: "コマンドラインオプション"
---

一貫性を保つため、コマンドラインインターフェース (CLI) フラグは `dbt` プレフィックスとそのサブコマンドの直後に記述する必要があります。これには「グローバル」フラグ（すべてのコマンドでサポートされます）も含まれます。設定可能なすべての dbt CLI フラグのリストについては、[使用可能なフラグ](/reference/global-configs/about-global-configs#available-flags) を参照してください。CLI フラグを設定すると、[環境変数](/reference/global-configs/environment-variable-configs) と [プロジェクトフラグ](/reference/global-configs/project-flags) がオーバーライドされます。

環境変数には `DBT_` プレフィックスが含まれます。

例えば、次のように記述する代わりに:

```bash
dbt --no-populate-cache run
```

こうすべきです:

```bash
dbt run --no-populate-cache
```

従来、サブコマンドの前にフラグ（「グローバルフラグ」など）を渡すことはレガシー機能であり、dbt Labs はいつでもこれを削除できます。サブコマンドの前後で同じフラグを使用することはサポートされていません。

## ブール型フラグと非ブール型フラグの使用

ブール型フラグを使用してコマンドを有効化または無効化したり、文字列などの特定の値を使用する非ブール型フラグを使用してコマンドを構成したりできます。

<Tabs>

<TabItem value="nonboolean" label="Non-boolean config">

以下の非ブール型構成構造を使用します。
- `<SUBCOMMAND>` を、この構成が適用されるコマンドに置き換えます。
- `<THIS-CONFIG>` を、有効化または無効化する構成に置き換えます。
- `<SETTING>` を、構成の新しい設定に置き換えます。

<File name='CLI flags'>


```text

<SUBCOMMAND> --<THIS-CONFIG>=<SETTING> 

```

</File>

### 例

<File name='CLI flags'>


```text

dbt run --printer-width=80 
dbt test --indirect-selection=eager

```

</File>

</TabItem>

<TabItem value="boolean" label="Boolean config">

ブール型設定を有効化または無効化するには、以下の手順に従います。
- この設定を適用する `<SUBCOMMAND>` を使用します。
- 有効にするには `--<THIS-CONFIG>` を、無効にするには `--no-<THIS-CONFIG>` を続けます。
- `<THIS-CONFIG>` を、有効化または無効化する設定に置き換えます。

<File name='CLI flags'>


```text
dbt <SUBCOMMAND> --<THIS-CONFIG> 
dbt <SUBCOMMAND> --no-<THIS-CONFIG> 

```

</File>

### 例

<File name='CLI flags'>


```text

dbt run --version-check
dbt run --no-version-check 

```

</File>

</TabItem>

</Tabs>
