SQL のベスト プラクティスとして、データをクリーンアップするロジックとデータを変換するロジックを分離する必要があります。既存のクエリでは、共通テーブル式 (CTE) を使用して既にこの作業を開始しています。

ロジックを個別のモデルに分離し、[ref](/reference/dbt-jinja-functions/ref) 関数を使用して他のモデルの上にモデルを構築することで、実験を行うことができます。

<Lightbox src="/img/dbt-dag.png" title="The DAG we want for our dbt project" />
