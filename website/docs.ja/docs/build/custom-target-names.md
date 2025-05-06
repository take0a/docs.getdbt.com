---
title: "カスタムターゲット名"
id: "custom-target-names"
description: "dbt プロジェクトの設定に対応するように、任意の dbt Cloud ジョブにカスタム ターゲット名を定義できます。"
pagination_next: null
---

## dbt Cloud Scheduler

dbt Cloud ジョブには、dbt プロジェクトの設定に合わせてカスタムターゲット名を定義できます。これは、dbt プロジェクト内に、指定したターゲットに応じて異なる動作をするロジックがある場合に役立ちます。例えば、次のようになります:

```sql
select *
from a_big_table

-- limit the amount of data queried in dev
{% if target.name != 'prod' %}
where created_at > date_trunc('month', current_date)
{% endif %}
```

dbt Cloud でジョブのカスタム ターゲット名を設定するには、Job Settings ページでジョブの **Target Name** フィールドを構成します。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/jobs-settings-target-name.png" title="Overriding the target name to 'prod'"/>

## dbt Cloud IDE

dbt Cloud で開発する場合、開発認証情報にカスタムターゲット名を設定できます。左パネルのプロフィールアイコンの上にあるアカウント名をクリックし、profile icon in the left panel, select **Account settings** を選択してから、**Credentials** に進みます。ターゲット名を更新するプロジェクトを選択してください。

<Lightbox src="/img/docs/dbt-cloud/using-dbt-cloud/development-credentials.png" title="Overriding the target name to 'dev'"/>
