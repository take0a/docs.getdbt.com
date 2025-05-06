---
title: Jinja をデバッグするにはどうすればいいですか?
description: "ターゲットフォルダまたはログ機能を使用してJinjaをデバッグする"
sidebar_label: 'Jinjaのデバッグ'
id: debugging-jinja

---

`target/compiled/<your_project>/` にあるコンパイル済みの SQL と `logs/dbt.log` にあるログを確認して、dbt がバックグラウンドで何を実行しているかを確認する方法に慣れておく必要があります。

[log](/reference/dbt-jinja-functions/log) 関数を使用して、オブジェクトをコマンドラインに出力することで Jinja をデバッグすることもできます。
