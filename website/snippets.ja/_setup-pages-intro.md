
<ul>
    <li><strong>Maintained by</strong>: {props.meta.maintained_by}</li>
    <li><strong>Authors</strong>: {props.meta.authors}</li>
    <li><strong>GitHub repo</strong>: <a href={`https://github.com/${props.meta.github_repo}`}>{props.meta.github_repo}</a>   <a href={`https://github.com/${props.meta.github_repo}`}><img src={`https://img.shields.io/github/stars/${props.meta.github_repo}?style=for-the-badge`}/></a></li>
    <li><strong>PyPI package</strong>: <code>{props.meta.pypi_package}</code> <a href={`https://badge.fury.io/py/${props.meta.pypi_package}`}><img src={`https://badge.fury.io/py/${props.meta.pypi_package}.svg`}/></a></li>
    <li><strong>Slack channel</strong>: <a href={props.meta.slack_channel_link}>{props.meta.slack_channel_name}</a></li>
    <li><strong>Supported dbt Core version</strong>: {props.meta.min_core_version} and newer</li>
    <li><strong>dbt Cloud support</strong>: {props.meta.cloud_support}</li>
    <li><strong>Minimum data platform version</strong>: {props.meta.min_supported_version}</li>
    </ul>

<h2> Installing {props.meta.pypi_package}</h2>

アダプターをインストールするには、`pip` を使用します。1.8 より前は、アダプターをインストールすると、`dbt-core` と追加の依存関係が自動的にインストールされていました。1.8 以降では、アダプターをインストールしても `dbt-core` は自動的にインストールされません。これは、アダプターと dbt Core バージョンが互いに分離されているため、既存の dbt-core インストールを上書きしたくないためです。
インストールには次のコマンドを使用します:
<VersionBlock firstVersion="1.8">

<code>python -m pip install dbt-core {props.meta.pypi_package}</code>

</VersionBlock>

<VersionBlock lastVersion="1.7">

<code>python -m pip install {props.meta.pypi_package}</code>

</VersionBlock>

<h2> Configuring {props.meta.pypi_package} </h2>

<p>For {props.meta.platform_name}-specific configuration, please refer to <a href={props.meta.config_page}>{props.meta.platform_name} configs.</a> </p>

