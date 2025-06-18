次の表は、ダウンストリーム エクスポージャーの視覚化とオーケストレーションの違いをまとめたものです。

| Info | 下流のエクスポージャーを設定して視覚化する | 下流のエクスポージャーを調整する <Lifecycle status="beta"/> |
| ---- | ---- | ---- |
| 目的 | ダウンストリーム アセットを dbt 系統に自動的に取り込みます。 | スケジュールされた dbt ジョブ中に、基礎となるデータ ソースをプロアクティブに更新します。 |
| 利点 | データ フローと依存関係を可視化します。 | 手動による介入なしに、BI ツールが常に最新のデータを持つことを保証します。 |
| 位置  | dbt [<Constant name="explorer"/>](/docs/explore/explore-projects) で公開 | [<Constant name="cloud" /> Scheduler](/docs/deploy/deployments) で公開 |
| サポートされているBIツール | Tableau | Tableau |
| ユースケース | ユーザーがモデルの使用方法を理解し、インシデントを削減するのに役立ちます。 | 必要に応じてモデルを実行することで、適時性を最適化し、コストを削減します。 |

ダウンストリームのエクスポージャーの視覚化と調整の詳細については、次のセクションを参照してください。

<div className="grid--2-col">

<Card
    title="下流のエクスポージャーを設定して視覚化する"
    body="ダッシュボードからダウンストリームのエクスポージャを自動的に設定し、ダウンストリーム ツールでモデルがどのように使用されているかを把握して、より豊富なダウンストリーム リネージを実現します。"
    link="/docs/cloud-integrations/downstream-exposures-tableau"
    icon="dbt-bit"/>

<Card
    title="下流のエクスポージャーを調整する"
    link="/docs/cloud-integrations/orchestrate-exposures"
    body="スケジュールされた dbt ジョブ中に dbt スケジューラを使用して、基礎となるデータ ソース (Tableau 抽出など) をプロアクティブに更新します。"
    icon="dbt-bit"/>

</div>
