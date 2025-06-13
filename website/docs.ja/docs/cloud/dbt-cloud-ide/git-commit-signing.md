---
title: "Git commit signing"
description: "Learn how to sign your Git commits when using the IDE for development."
sidebar_label: Git commit signing
---

# Git コミット署名 <Lifecycle status="managed,managed_plus" />

なりすましを防ぎ、セキュリティを強化するために、<Constant name="git" /> コミットをリポジトリにプッシュする前に署名することができます。署名を使用することで、<Constant name="git" /> プロバイダーはコミットを暗号的に検証し、「検証済み」としてマークできるため、コミットの出所に関する信頼性が向上します。

開発に <Constant name="cloud_ide" /> を使用する場合、<Constant name="cloud" /> を設定して <Constant name="git" /> コミットに署名することができます。設定するには、<Constant name="cloud" /> でこの機能を有効にし、フローに従ってキーペアを生成し、署名検証に使用する公開鍵を <Constant name="git" /> プロバイダーにアップロードします。


## 前提条件

- GitHub または GitLab が <Constant name="git" /> プロバイダーであること。現在、Azure DevOps はサポートされていません。
- [Enterprise プランまたは Enterprise+ プラン](https://www.getdbt.com/pricing/) の <Constant name="cloud" /> アカウントをお持ちであること。

## dbt で GPG キーペアを生成する

<Constant name="cloud" /> で GPG キーペアを生成するには、以下の手順に従います。
1. <Constant name="cloud" /> の **Personal profile** ページに移動します。
2. **Signed Commits** セクションに移動します。
3. **Sign commits originating from this user** トグルを有効にします。
4. これで GPG キーペアが生成されます。秘密鍵は、今後のすべての <Constant name="git" /> コミットの署名に使用されます。公開鍵が表示されるので、<Constant name="git" /> プロバイダーにアップロードできます。

<Lightbox src="/img/docs/dbt-cloud/example-git-signed-commits-setting.png" width="95%" title="Example of profile setting Signed commits" />

## Git プロバイダーに公開鍵をアップロードする

<Constant name="git" /> プロバイダーに公開鍵をアップロードするには、サポートされている <Constant name="git" /> プロバイダーが提供する詳細なドキュメントに従ってください。

- [GitHub instructions](https://docs.github.com/en/authentication/managing-commit-signature-verification/adding-a-gpg-key-to-your-github-account) 
- [GitLab instructions](https://docs.gitlab.com/ee/user/project/repository/signed_commits/gpg.html) 

公開キーを <Constant name="git" /> プロバイダーにアップロードすると、変更をリポジトリにプッシュした後、 <Constant name="git" /> コミットは「検証済み」としてマークされます。

<Lightbox src="/img/docs/dbt-cloud/git-sign-verified.jpg" width="95%" title="Example of a verified Git commit in a Git provider." />

## 考慮事項

- GPG キーペアは特定のアカウントではなく、ユーザーに紐付けられます。ユーザーとキーペアは 1:1 の関係です。ユーザーが所属するすべてのアカウントのコミットの署名には、同じキーが使用されます。
- <Constant name="cloud" /> で生成される GPG キーペアは、キーペア作成時にアカウントに関連付けられたメールアドレスにリンクされます。このメールアドレスは、署名されたコミットの作成者を識別します。
- <Constant name="git" /> コミットを「検証済み」とマークするには、<Constant name="cloud" /> メールアドレスが、<Constant name="git" /> プロバイダで検証済みのメールアドレスである必要があります。<Constant name="git" /> プロバイダ（GitHub、GitLab など）は、コミットの署名済みメールアドレスが、<Constant name="git" /> プロバイダアカウントで検証済みのメールアドレスと一致するかどうかを確認します。一致しない場合、コミットは「検証済み」とマークされません。
- 検証の問題を回避するため、<Constant name="cloud" /> のメールアドレスと <Constant name="git" /> プロバイダの確認済みメールアドレスを同期させておいてください。<Constant name="cloud" /> のメールアドレスを変更する場合は、以下の手順に従ってください。
  - [前述の手順](/docs/cloud/dbt-cloud-ide/git-commit-signing#generate-gpg-keypair-in-dbt-cloud) に従って、更新したメールアドレスで新しい GPG キーペアを生成します。
  - <Constant name="git" /> プロバイダに新しいメールアドレスを追加して検証します。

<!-- vale off -->

## FAQs

<!-- vale on -->

<DetailsToggle alt_header="dbt で GPG キーペアを削除するとどうなりますか?">

<Constant name="cloud" /> で GPG キーペアを削除すると、Git コミットは署名されなくなります。[前述の手順](/docs/cloud/dbt-cloud-ide/git-commit-signing#generate-gpg-keypair-in-dbt-cloud) に従って新しい GPG キーペアを生成できます。
</DetailsToggle>

<DetailsToggle alt_header="どの Git プロバイダーが GPG キーをサポートしていますか?">

GitHubとGitLabはコミット署名をサポートしていますが、Azure DevOpsはサポートしていません。コミット署名は[gitの機能](https://git-scm.com/book/ms/v2/Git-Tools-Signing-Your-Work)であり、特定のプロバイダーに依存しません。ただし、すべてのプロバイダーが公開鍵のアップロードやコミット時の検証バッジの表示をサポートしているわけではありません。

</DetailsToggle>

<DetailsToggle alt_header="Git プロバイダーが GPG キーをサポートしていない場合はどうなりますか?">

Git プロバイダーが公開 GPG キーのアップロードを明示的にサポートしていない場合、コミットは秘密キーを使用して署名されますが、プロバイダーによって検証情報は表示されません。

</DetailsToggle>

<DetailsToggle alt_header="Git プロバイダーがすべてのコミットに署名を要求する場合はどうなりますか?">

Git プロバイダーがコミット検証を強制するように設定されている場合、署名されていないコミットは拒否されます。これを回避するには、前の手順をすべて実行してキーペアを生成し、公開鍵をプロバイダーにアップロードしてください。

</DetailsToggle>
