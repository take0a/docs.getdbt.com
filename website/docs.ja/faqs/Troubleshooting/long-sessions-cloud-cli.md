---
title: "dbt Cloud CLI で「Session occupied」というエラーが表示されます。"
description: "dbt Cloud CLI で長時間実行セッションをデバッグする方法"
sidebar_label: 'dbt Cloud CLI で長時間実行セッションをデバッグする'
id: long-sessions-cloud-cli
---

dbt Cloud CLI で「セッションが占有されています」というエラーが表示される場合、またはセッションが長時間実行されている場合は、別のターミナルウィンドウで「dbt invocation list」コマンドを使用して、アクティブなセッションのステータスを確認できます。これは、問題をデバッグし、セッションの長時間実行の原因となっている引数を特定するのに役立ちます。

アクティブなセッションをキャンセルするには、「Ctrl + Z」ショートカットを使用します。

「dbt invocation」コマンドの詳細については、[dbt invocation コマンド リファレンス](/reference/commands/invocation) をご覧ください。

または、<code>dbt reattach</code> を使用して既存のセッションに再接続し、<code>Control-C</code> キーを押して呼び出しをキャンセルすることもできます。
