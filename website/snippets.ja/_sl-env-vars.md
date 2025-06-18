セマンティックレイヤーに接続するときは、[拡張属性](/docs/dbt-cloud-environments#extended-attributes)と[環境変数](/docs/build/environment-variables)を使用します。セマンティックレイヤー認証情報に直接値を設定すると、拡張属性よりも優先されます。環境変数を使用する場合は、環境のデフォルト値が使用されます。

たとえば、セマンティックレイヤー認証情報で `{{env_var('DBT_WAREHOUSE')}}` を使用してウェアハウスを設定します。

同様に、拡張属性で `{{env_var('DBT_ACCOUNT')}}` を使用してアカウントの値を設定すると、dbt は拡張属性と環境変数の両方をチェックします。
