import SLEnvVars from '/snippets.ja/_sl-env-vars.md';

環境レベルおよびプロジェクトレベルでセマンティックレイヤーを設定するには、オーナーグループに属し、適切な[ライセンス](/docs/cloud/manage-access/seats-and-users)と[権限](/docs/cloud/manage-access/enterprise-permissions)を持っている必要があります。
- Enterprise+ および Enterprise プラン：
   - アカウント管理者権限を持つ開発者ライセンス、または
   - 開発者ライセンスを持ち、プロジェクト作成者、データベース管理者、または管理者権限が割り当てられたオーナー。
- Starter プラン：開発者ライセンスを持つオーナー。
- 無​​料トライアル：現在、Starter プランの無料トライアルをオーナーとして利用しています。つまり、dbt セマンティックレイヤーにアクセスできます。

### 1. Select environment

セマンティックレイヤーを有効にする環境を選択します。

1. ナビゲーションメニューの「**アカウント設定**」に移動します。
2. 左側のサイドバーの「**設定**」で、セマンティックレイヤーを有効にするプロジェクトを選択します。
3. 「**プロジェクトの詳細**」ページで、「**セマンティックレイヤー**」セクションに移動します。「**セマンティックレイヤーを構成**」を選択します。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/new-sl-configure.jpg" width="70%" title="Semantic Layer section in the 'Project Details' page"/>

4. **「セマンティック レイヤー構成の設定」** ページで、セマンティック レイヤーを適用する環境を選択し、「保存」をクリックします。これにより、管理者はセマンティック レイヤーを有効にする環境を柔軟に選択できるようになります。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-select-env.jpg" width="75%" title="Select the deployment environment to run your Semantic Layer against."/>

### 2. Add a credential and create service tokens

dbt セマンティック レイヤーは、認証に [サービス トークン](/docs/dbt-cloud-apis/service-tokens) を使用します。このトークンは、ユーザーが設定した基盤となるデータ プラットフォームの認証情報に紐付けられています。設定された認証情報は、セマンティック レイヤーがデータ プラットフォームに対して発行するクエリの実行に使用されます。

この認証情報は、セマンティック レイヤーがアクセスする基盤となるデータへの物理的なアクセスを制御し、データ プラットフォームでこの認証情報に対して設定されているすべてのアクセス ポリシーが適用されます。

| Feature | Starter plan | Enterprise+ and Enterprise plan |
| --- | --- | --- |
| サービストークン | 1つの認証情報にリンクされた複数のサービストークンを作成できます。 | 複数の認証情報を使用し、各認証情報に複数のサービストークンをリンクできます。ただし、1つのサービストークンを複数の認証情報にリンクすることはできません。 |
| プロジェクトごとの認証情報 | プロジェクトごとに 1 つの認証情報。 | プロジェクトごとに複数の認証情報を追加できます (#4-add-more-credentials)。 |
| 複数のサービス トークンを 1 つの認証情報にリンクする | ✅ | ✅ |

*スタータープランをご利用で、認証情報を追加する必要がある場合は、[Enterprise+プランまたはEnterpriseプラン](https://www.getdbt.com/contact)へのアップグレードをご検討ください。Enterpriseプランをご利用のお客様は、[認証情報の追加](#4-add-more-credentials)で複数の認証情報を追加する手順の詳細をご確認ください。*

#### 1. デプロイ環境の選択
- デプロイ環境を選択すると、「**認証情報とサービストークン**」ページが表示されます。
- 「**セマンティックレイヤー認証情報を追加**」ボタンをクリックします。

#### 2. 認証情報の設定
- **1. 認証情報の追加** セクションで、セマンティックレイヤーで使用するデータプラットフォーム固有の認証情報を入力します。
- 最小限の権限を持つ認証情報を使用してください。セマンティックレイヤーには、下流アプリケーションのセマンティックモデルで使用される dbt モデルを含むスキーマへの読み取りアクセス権が必要です。
- <SLEnvVars/>

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-add-credential.jpg" width="55%" title="Add credentials and map them to a service token. " />

#### 3. Create or link service tokens
- サービストークンを作成する権限がある場合は、認証情報を追加した後、[**新しいサービストークンをマップする** オプション](/docs/use-dbt-semantic-layer/setup-sl#map-service-tokens-to-credentials)が表示されます。トークンに名前を付け、権限を「セマンティックレイヤーのみ」と「メタデータのみ」に設定して、[**保存**] をクリックします。
- トークンが生成されると、このトークンを再度表示できなくなりますので、安全な場所に記録しておいてください。
- サービストークンを作成する権限がない場合は、管理者に連絡してトークンを作成するように求めるメッセージが表示されます。管理者は必要に応じてトークンを作成し、リンクできます。
   <Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-credential-no-service-token.jpg" width="70%" title="If you don’t have access to create service tokens, you can create a credential and contact your admin to create one for you." />

:::info
- スタータープランでは、単一の基盤となる認証情報にリンクする複数のサービストークンを作成できますが、各プロジェクトに設定できる認証情報は1つだけです。
- すべてのエンタープライズプランでは、[複数の認証情報を追加](#4-add-more-credentials)し、それらをサービストークンにマッピングして、アクセスをカスタマイズできます。

   <a href="https://www.getdbt.com/contact" style={{ color: 'white', backgroundColor: '#66c2c2', padding: '4px 8px', borderRadius: '4px', textDecoration: 'none', display: 'inline-block' }}>無料のライブデモを予約</a>して、<Constant name="cloud" /> エンタープライズプラン以上の可能性を最大限にご体験ください。
:::

### 3. View connection detail
1. 下流ツールに接続するための接続情報を確認するには、**プロジェクトの詳細** ページに戻ります。
2. 環境ID、サービストークン、ホスト、およびサービストークン名をコピーし、BI接続設定を担当するチームに共有します。ツールでGraphQL APIを使用している場合は、JDBC URLではなくGraphQL APIホスト情報を保存してください。

他の統合への接続方法については、[利用可能な統合](/docs/cloud-integrations/avail-sl-integrations)を参照してください。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-configure-example.jpg" width="50%" title="After configuring, you'll be provided with the connection details to connect to you downstream tools." />

### 4. Add more credentials <Lifecycle status="managed_plus,managed" />
すべての <Constant name="cloud" /> Enterprise プランでは、オプションで複数の認証情報を追加し、それらをサービストークンにマッピングできます。これにより、よりきめ細かな制御と、各チームに合わせたアクセスが可能になります。これらの認証情報は、BI 接続設定時に関連チームと共有できます。これらの認証情報は、セマンティックレイヤーがアクセスする基盤データへの物理アクセスを制御します。

認証情報とサービストークンは、チームとその役割に合わせて設定することをお勧めします。例えば、財務チームに財務関連のスキーマへのアクセスを提供するなど、チームのニーズに合わせたトークンまたは認証情報を作成します。

<Expandable alt_header="資格情報のリンクに関する考慮事項">

- 管理者はプロジェクト内の 1 つの認証情報に複数のサービス トークンをリンクできますが、各サービス トークンはプロジェクトごとに 1 つの認証情報にのみリンクできます。
- API 経由でリクエストを送信すると、リンクされた認証情報のサービス トークンは、セマンティック レイヤー リクエストの構築に使用される基礎となるビューとテーブルのアクセス ポリシーに従います。
- <SLEnvVars/>
</Expandable>

#### 1. 認証情報を追加する
- 環境を設定したら、「**認証情報とサービストークン**」ページで「**セマンティックレイヤー認証情報を追加**」ボタンをクリックして複数の認証情報を作成し、サービストークンにマッピングします。<br />
- 「**1. 認証情報を追加**」セクションで、データプラットフォームの認証情報フィールドに入力します。「読み取り専用」の認証情報を使用することをお勧めします。
   <Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-add-credential.jpg" width="55%" title="Add credentials and map them to a service token. " />

#### 2. サービストークンを認証情報にマッピングする
- 「**2. 新しいサービストークンをマッピングする**」セクションで、前の手順で構成した[サービストークンを認証情報にマッピング](/docs/use-dbt-semantic-layer/setup-sl#map-service-tokens-to-credentials)します。<Constant name="cloud" /> により、必要なサービストークン権限セット（セマンティックレイヤーのみとメタデータのみ）が自動的に選択されます。
- 構成中に別のサービストークンを追加するには、「**サービストークンを追加**」をクリックします。
- 後で「**セマンティックレイヤー構成の詳細**」ページで、同じ認証情報に複数のサービストークンをリンクできます。既存のセマンティックレイヤー構成に別のサービストークンを追加するには、「**リンクされたサービストークン**」セクションの「**サービストークンを追加**」をクリックします。
- 「**保存**」をクリックして、サービストークンを認証情報にリンクします。サービストークンは生成後に再度表示できないため、必ずコピーして安全に保存してください。
<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-credentials-service-token.jpg" width="90%" title="Use the configuration page to manage multiple credentials or link or unlink service tokens for more granular control."/>

#### 3. 認証情報を削除する
- 認証情報を削除するには、「**認証情報とサービストークン**」ページに戻ります。
- 「**リンクされたサービストークン**」で「**編集**」をクリックし、「**認証情報の削除**」を選択して認証情報を削除します。

   認証情報を削除すると、プロジェクト内でその認証情報にマッピングされているすべてのサービストークンが機能しなくなり、エンドユーザーにとって使用できなくなります。

### 設定の削除
プロジェクトのセマンティック レイヤー設定全体を削除できます。セマンティック レイヤー設定を削除すると、すべての認証情報が削除され、すべてのサービス トークンとプロジェクトのリンクが解除されます。また、セマンティック レイヤーへのすべてのクエリが失敗します。

プロジェクトのセマンティック レイヤー設定を削除するには、以下の手順に従ってください。

1. **プロジェクトの詳細** ページに移動します。
2. **セマンティック レイヤー** セクションで、**セマンティック レイヤーの削除** を選択します。
3. 確認ポップアップで [はい、セマンティック レイヤーを削除します] をクリックして削除を確定します。

今後、dbt セマンティック レイヤー設定を再度有効にするには、[前の手順](#set-up-dbt-semantic-layer) に従って設定を再作成する必要があります。セマンティック モデルと指標がプロジェクト内にまだ残っている場合は、変更は必要ありません。削除した場合は、YAML 設定を再度設定する必要があります。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-delete-config.jpg" width="90%" title="Delete the Semantic Layer configuration for a project."/>

## 追加設定

   以下は、セマンティックレイヤー認証情報に関する追加の柔軟な設定です。

### サービストークンを認証情報にマッピングする
- 環境の設定後、必要な[権限](/docs/cloud/manage-access/about-user-access#permission-sets)があれば、同じ認証情報に追加のサービストークンをマッピングできます。
- **認証情報とサービストークン** ページに移動し、**リンクされたサービストークン** セクションの **+ サービストークンを追加** ボタンをクリックします。
- サービストークン名を入力し、必要な権限セット（セマンティックレイヤーのみとメタデータのみ）を選択します。
- **保存** をクリックして、サービストークンを認証情報にリンクします。
- サービストークンは生成後に再度表示できないため、必ずコピーして安全に保存してください。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-add-service-token.gif" title="Map additional service tokens to a credential." />

### サービストークンのリンクを解除する
- 「**リンクされたサービストークン**」セクションの「**リンク解除**」をクリックして、サービストークンと認証情報のリンクを解除します。リンクされていない認証情報でセマンティックレイヤーにクエリを実行しようとすると、有効なトークンがマッピングされていないため、BIツールでエラーが発生します。

### サービストークンページから管理
**サービストークンから認証情報を表示**
- **APIトークン**、そして**サービストークン**ページに移動すると、セマンティックレイヤーの認証情報を直接確認できます。
- サービストークンを選択すると、リンクされている認証情報が表示されます。これは、プロジェクト内の認証情報にマッピングされているサービストークンを確認したい場合に便利です。

#### 新しいサービストークンを作成する
- **サービストークン** ページから新しいサービストークンを作成し、認証情報にマッピングします（セマンティックレイヤー権限が存在することを前提としています）。これは、新しいサービストークンを作成し、それをプロジェクト内の認証情報に直接マッピングする場合に便利です。
- サービストークンの適切な権限セット（セマンティックレイヤーのみとメタデータのみ）を選択してください。

<Lightbox src="/img/docs/dbt-cloud/semantic-layer/sl-create-service-token-page.jpg" width="100%" title="Create a new service token and map credentials directly on the separate 'Service tokens page'."/>
