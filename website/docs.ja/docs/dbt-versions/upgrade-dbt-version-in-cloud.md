---
title: "Upgrade dbt version in Cloud"
id: "upgrade-dbt-version-in-cloud"
---

<Constant name="cloud" /> では、[ジョブ](/docs/deploy/jobs) と [環境](/docs/dbt-cloud-environments) の両方が、特定のバージョンの <Constant name="core" /> を使用するように設定されています。バージョンはいつでもアップグレードできます。

## 環境

環境の設定ページに移動し、**編集** をクリックします。**dbt バージョン** のドロップダウンバーをクリックして選択します。[リリーストラック](#release-tracks) を選択して継続的なアップデートを受け取るか（推奨）、<Constant name="core" /> のレガシーバージョンを選択できます。変更内容は必ず保存してから別のページに移動してください。

<Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/choosing-dbt-version/example-environment-settings.png" width="90%" title="Example environment settings in dbt"/>

### リリーストラック

2024年以降、プロジェクトはお客様が選択した頻度で自動的にアップグレードされます。

**最新** トラックでは、最新の <Constant name="cloud" /> 機能と、dbt フレームワークの新機能への早期アクセスが保証されます。**互換** トラックと **拡張** トラックは、リリース頻度の低いリリース、本番環境への導入前に新しい dbt リリースをテストする機能、または <Constant name="core" /> の最新のオープンソースリリースとの継続的な互換性を必要とするお客様向けに設計されています。

dbt Labs では、ベストプラクティスとして、まず開発環境でアップグレードをテストすることを推奨しています。[dbt バージョンのオーバーライド](#override-dbt-version) 設定を使用して、デプロイメント環境とすべての同僚のデフォルトの開発環境をアップグレードする前に、最新の dbt バージョンでプロジェクトをテストしてください。

[<Constant name="cloud" /> Admin API](/docs/dbt-cloud-apis/admin-cloud-api) または [Terraform](https://registry.terraform.io/providers/dbt-labs/dbtcloud/latest) で環境をアップグレードするには、`dbt_version` をリリーストラックの名前に設定します。
- `latest` (以前は `versionless` と呼ばれていましたが、古い名前も引き続きサポートされています)
- `compatible` (Starter、Enterprise、Enterprise+ プランで利用可能)
- `extended` (すべての Enterprise プランで利用可能)

### dbt バージョンのオーバーライド

[開発環境](/docs/dbt-cloud-environments#types-of-environments) で設定されているものとは異なる dbt バージョンを使用するようにプロジェクトを設定します。このオーバーライドは、自分のユーザーアカウントにのみ影響し、他のユーザーのアカウントには影響しません。プロジェクトの dbt バージョンをアップグレードする前に、このオーバーライドを使用して新しい dbt 機能を安全にテストしてください。

1. 左側のサイドパネルでアカウント名をクリックし、[アカウント設定] を選択します。
2. サイドバーから [認証情報] を選択し、プロジェクトを選択します。サイドパネルが開きます。
3. サイドパネルで [編集] をクリックし、[ユーザー開発設定] セクションまでスクロールします。
4. [dbt バージョン] ドロップダウンからバージョンを選択し、[保存] をクリックします。

    選択したプロジェクトの構成済みバージョンを [「最新」リリース トラック](/docs/dbt-versions/cloud-release-tracks) にオーバーライドする例:

  <Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/choosing-dbt-version/example-override-version.png" width="60%" title="Example of overriding the dbt version on your user account"/>

5. （オプション）<Constant name="cloud" /> のコマンドバーで `dbt build` コマンドを実行し、<Constant name="cloud_ide" /> がオーバーライド設定を使用してプロジェクトをビルドすることを確認します。**システムログ** セクションを展開し、出力の最初の行を見つけます。`Running with dbt=` で始まり、<Constant name="cloud" /> が使用しているバージョンがリストされているはずです。<br /><br />
    リリーストラックのユーザーの場合、出力には特定のバージョンではなく `Running dbt...` と表示されます。これは、リリーストラック機能によって提供される柔軟性と継続的な自動更新を反映しています。

## ジョブ

<Constant name="cloud" /> 内の各ジョブは、所属する環境からパラメータを継承するように設定できます。

<Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/choosing-dbt-version/job-settings.png" width="200%" title="Settings of a dbt job"/>

上記のスクリーンショットに示されているサンプルジョブは、環境「Prod」に属しています。**ENVIRONMENT_NAME (DBT_VERSION) から継承** が選択されていることからわかるように、環境の dbt バージョンを継承しています。また、ドロップダウンから別のオプションを選択することで、特定のジョブの dbt バージョンを、Cloud でサポートされている現在の Core リリースのいずれかに手動で上書きすることもできます。

## サポート対象バージョン

dbt Labs は、新しいマイナーバージョンがリリースされるたびに、dbt Core のバージョンアップグレードをユーザーに推奨してきました。2021年12月に、dbt の最初のメジャーバージョンである「dbt 1.0」をリリースしました。このリリースに合わせて、<Constant name="dbt_platform" /> でサポートする dbt Core のバージョンに関するポリシーを更新しました。

> **v1.0 以降、以降のすべてのマイナーバージョンは <Constant name="cloud" /> でご利用いただけます。各バージョンは、最初のリリースから1年間、パッチとバグ修正を含むアクティブなサポート期間が提供されます。1年間の期間終了後は、継続的なメンテナンスとサポートの向上のため、すべてのユーザーに新しいバージョンへのアップグレードを推奨します。**

バージョンごとに異なるサポートレベルを提供しており、これには新機能、バグ修正、セキュリティパッチなどが含まれる場合があります。

<Snippet path="core-version-support" />

<Constant name="cloud" /> で Core のさまざまなバージョンのサポートを停止する予定時期をユーザーが把握できるように、次のリリース テーブルを継続的に更新します。

<Snippet path="core-versions-table" />

v1.0以降、<Constant name="cloud" /> を使用すると、最新の修正を含む `dbt-core` とプラグインの最新の互換性のあるパッチリリースを常に使用できるようになります。また、これらのパッチリリースのプレリリース版を一般公開前に試用することもできます。

<!--- TODO: Include language to reflect:
  - notifying users when new minor versions are available
  - notifying users when using a minor version that is nearing the end of its critical support period
  - auto-upgrading users to the subsequent minor version when critical support ends
--->

バージョンのサポートと将来のリリースの詳細については、[<Constant name="core" /> バージョンについて](/docs/dbt-versions/core) を参照してください。

### dbt Fusion エンジン

dbt Labs は、dbt を根本から再構築した新しい [dbt Fusion エンジン](/docs/fusion/about-fusion) を導入しました。これは現在、dbt プラットフォーム上でベータ版として提供されています。対象となるお客様は、v1.x と同じワークフローを使用して環境を Fusion にアップデートできますが、いくつか注意事項があります。
- **Fusion 最新リリーストラックにアクセスするには、dbt Labs アカウントチームにリクエストを送信する必要があります。ベータ版の対象範囲は、スタータープランを含むプロジェクトの要件に基づいて、毎週拡大していきます**。ベータ版からプレビュー版に移行すると、すべてのユーザーの環境、プロジェクト、ジョブなどで Fusion がオプションとして表示されます。

プロジェクトの互換性を高めるには、すべてのジョブと環境を「最新」リリーストラックにアップデートし、[アップグレードガイド](/docs/dbt-versions/core-upgrade/upgrading-to-fusion) に従ってください。
- いくつか重要な変更点があります。詳細は[アップグレードガイド](/docs/dbt-versions/core-upgrade/upgrading-to-fusion)でもご確認いただけます。
- 現在サポートされているアダプタはSnowflakeのみです。今後、さらに多くのアダプタのサポートが追加される予定です。
- 開発環境を「Fusion Latest」に変更すると、すべてのユーザーがIDEを再起動する必要があります。


  <Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/cloud-upgrading-dbt-versions/upgrade-fusion.png" width="90%" title="Upgrade to the Fusion engine in your environment settings." />


### アップグレードについてサポートが必要ですか？

dbt プロジェクトのアップグレード方法についてさらに詳しいアドバイスが必要な場合は、[移行ガイド](/docs/dbt-versions/core-upgrade/)と[アップグレードに関する Q&A ページ](/docs/dbt-versions/upgrade-dbt-version-in-cloud#upgrading-legacy-versions-under-10)をご覧ください。


### アップグレード前の変更点のテスト

必要なコード変更がわかったら、実装を開始できます。以下の手順をお勧めします:

- メインの dbt プロジェクトに反映する前に、変更点をテストするための「アップグレード プロジェクト」という別の dbt プロジェクトを作成します。
- 「アップグレード プロジェクト」で、本番環境プロジェクトと同じリポジトリに接続します。
- 開発環境の [設定](/docs/dbt-versions/upgrade-dbt-version-in-cloud) を、<Constant name="core" /> の最新バージョンを実行するように設定します。
- ブランチ `dbt-version-upgrade` をチェックアウトし、プロジェクトに適切な更新を加え、<Constant name="cloud_ide" /> の新しいバージョンで dbt プロジェクトがコンパイルされ、実行されることを確認します。
  - 最新バージョンに直接アップグレードすると問題が多すぎる場合は、マイナーバージョンを順にアップグレードしてプロジェクトを反復的にテストしてみてください。 <Constant name="core" /> の遠いバージョン（例：1.0 → 1.10）間では、長年の開発期間といくつかの互換性のない変更が存在します。連続するマイナーバージョン間のアップグレードで問題が発生する可能性は大幅に低いため、定期的なアップグレードをお勧めします。
- `dbt-version-upgrade` ブランチの開発環境で、最新バージョンの dbt でプロジェクトをコンパイルして実行したら、本番環境ジョブの 1 つを複製してブランチのコードから実行してみてください。
- これを行うには、テスト用の新しいデプロイメント環境を作成し、カスタムブランチを「ON」に設定し、`dbt-version-upgrade` ブランチを参照します。また、この環境で dbt のバージョンを最新の dbt Core バージョンに設定する必要があります。

<Lightbox src="/img/docs/dbt-cloud/cloud-configuring-dbt-cloud/cloud-upgrading-dbt-versions/upgrade-environment.png" width="90%" title="Setting your testing environment" />

- 次に、チームが依存している本番環境ジョブの 1 つを複製するジョブを新しいテスト環境に追加します。
  - そのジョブがスムーズに実行された場合、ブランチをメインにマージする準備が整っているはずです。
  - 次に、メインの dbt プロジェクトの開発環境とデプロイメント環境を変更し、<Constant name="core" /> の最新バージョンで実行できるようにします。
