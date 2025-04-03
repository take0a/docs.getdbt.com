---
title: YAML のスタイル設定方法
id: 5-how-we-style-our-yaml
---

## YAML スタイルガイド

- 2️⃣ インデントは2スペースにしてください
- ➡️ リスト項目はインデントする必要がある
- 🔠 単一のエントリを持つリスト項目は文字列にすることができます。たとえば、`'select': 'other_user'` ですが、明示的なリストとして引数を指定するのがベストプラクティスです。たとえば、`'select': ['other_user']`
- 🆕 適切な場合は、辞書のリスト項目を新しい行で区切ってください。
- 📏 YAML の行は 80 文字以内にする必要があります。
- 🛠️ 互換性のある IDE と YAML フォーマッタ ([Prettier](https://prettier.io/) を併用) で [dbt JSON スキーマ](https://github.com/dbt-labs/dbt-jsonschema) を使用して、YAML ファイルを検証し、自動的にフォーマットします。

:::info
☁️ Python や SQL と同様に、dbt Cloud IDE には Prettier による YAML ファイル (Markdown と JSON も!) のフォーマット機能が組み込まれています。[Format] ボタンをクリックするだけで、完璧なスタイルになります。他のツールと同様に、[フォーマット ルールをカスタマイズ](https://docs.getdbt.com/docs/cloud/dbt-cloud-ide/lint-format#format-yaml-markdown-json) して、会社のスタイル ガイドに合うようにすることもできます。
:::

### YAML の例

```yaml
version: 2

models:
  - name: events
    columns:
      - name: event_id
        description: This is a unique identifier for the event
        tests:
          - unique
          - not_null

      - name: event_time
        description: "When the event occurred in UTC (eg. 2018-01-01 12:00:00)"
        tests:
          - not_null

      - name: user_id
        description: The ID of the user who recorded the event
        tests:
          - not_null
          - relationships:
              to: ref('users')
              field: id
```
