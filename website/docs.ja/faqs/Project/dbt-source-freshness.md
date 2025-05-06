---
title: Are the results of freshness stored anywhere?
description: "How to access Source Freshness results"
sidebar_label: 'Accessing Source Freshness results'
id: dbt-source-freshness

---
はい！

`dbt source freshness` コマンドは、フレッシュネススナップショットで選択された各 <Term id="table" /> について、pass/warning/error ステータスを出力します。

さらに、dbt はフレッシュネスの結果を、デフォルトで `target/` ディレクトリ内の `sources.json` というファイルに書き込みます。`dbt source freshness` コマンドに `-o` フラグを使用することで、この出力先をオーバーライドすることもできます。

ジョブ内でソースフレッシュネスを有効にした後、**Project Details** ページで [アーティファクト](/docs/deploy/artifacts) を構成します。このページは、dbt Cloud の左側のメニューでアカウント名を選択し、**Account settings** をクリックすると表示されます。ジョブページで **View Sources** をクリックすると、ソースフレッシュネスの現在のステータスを確認できます。
