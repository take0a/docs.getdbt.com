---
title: "dbt製品ライフサイクル"
id: "product-lifecycles"
description: "dbt Labs の製品ライフサイクルについて学びます。"
---

dbt Labs は、以下の 2 つの製品のメンテナンスに直接関与しています。

- <Constant name="core" />: The [open-source](https://github.com/dbt-labs/dbt-core) software that’s freely available.
- <Constant name="cloud" />: The cloud-based [SaaS solution](https://www.getdbt.com/signup), originally built on top of <Constant name="core" />. We're now introducing dbt's new engine, the <Constant name="fusion_engine" />. For more information, refer to [About the dbt Fusion engine](/docs/fusion/about-fusion).
- <Constant name="fusion_engine" />: The next-generation dbt engine, substantially faster than  <Constant name="core" /> and has built in SQL comprehension technology to power the next generation of analytics engineering workflows. The <Constant name="fusion_engine" /> is designed to deliver data teams a lightning-fast development experience, intelligent cost savings, and improved governance.

All dbt features fall into a lifecycle category determined by their availability in the following products:

### The dbt platform

<Constant name="cloud" /> の機能はすべて、以下のいずれかのカテゴリに分類されます。

- **ベータ版:** ベータ版機能は現在開発中であり、一部のお客様のみご利用いただけます。ベータ版への参加には、サインアップフォームが用意されている場合や、dbt Labs から特定のお客様にテストに関するご連絡を差し上げる場合があります。一部の機能は、アカウントで [試験的機能](/docs/dbt-versions/experimental-features) を有効にすると有効になります。ベータ版機能は未完成で、完全に安定していない可能性があります。互換性を破る変更が発生する可能性があるため、お客様の責任でご使用ください。ベータ版機能のドキュメントが完全に整備されていない可能性があり、テクニカルサポートが制限され、サービスレベル目標 (SLO) が提供されない場合があります。詳細については、[ベータ版機能の利用規約](/assets/beta-tc.pdf) をダウンロードしてください。
- **プレビュー版:** プレビュー版機能は安定しており、本番環境での導入に向けて機能的に準備が整っていると見なされています。一般公開される前に、機能の動作に関する計画的な追加や変更が行われる場合があります。また、下位互換性のない新機能が導入される可能性もあります。プレビュー機能には、ドキュメント、テクニカルサポート、サービスレベル目標 (SLO) が含まれます。プレビュー機能は追加料金なしで提供されますが、一般提供開始時には有料機能となる場合があります。
- **一般提供 (GA):** 一般提供機能は、条件を満たしたすべての <Constant name="cloud" /> アカウントに導入された安定した機能です。GA 機能には、ドキュメントやテクニカルサポートを含むサービスレベル契約 (SLA) が適用されます。一部の GA 機能が利用できるかどうかは、環境の dbt バージョンによって決まります。常に最新の GA 機能を利用するには、<Constant name="cloud" /> [環境](/docs/dbt-cloud-environments) がサポートされている [リリース トラック](/docs/dbt-versions/cloud-release-tracks) になっていることを確認してください。
- **非推奨:** この状態の機能は、dbt Labs による開発または拡張が終了しています。これらの機能はそのまま機能し続け、ドキュメントは削除日まで保持されます。ただし、テクニカルサポートは提供されなくなります。
- **削除済み:** 削除された機能は、プラットフォーム上でいかなる形でも利用できなくなります。

### dbt Core

<Constant name="core" /> は以下のライフサイクル状態でリリースされます。Core リリースはセマンティックバージョニングに従います。詳細については、[Core バージョンについて](/docs/dbt-versions/core) をご覧ください。

- **未リリース:** 次のマイナーバージョンのプレリリースにこの機能を含める予定です。ただし、その動作や実装については確約できません。メンテナーとして、この機能の一部を変更したり、（説明を添えて）完全に削除したりする権利を留保します。
- **プレリリース:**
   * **ベータ:** ベータ版の目的は、このマイナーバージョンが最終リリースされた際に導入される新機能の概要を最初に確認できるようにすることです。ベータ版に含まれるコードは、既存の機能からの回帰や、リリース済みの他の機能との悪影響なしに動作するはずです。ベータ版に含まれる新機能は、不完全な場合や、既知のエッジケース/制限事項がある場合があります。ベータ版に含まれる変更は「ロック」されておらず、メンテナーは変更または削除する権利を留保します（説明付き）。
   * **リリース候補版:** リリース候補版の目的は、最終リリースで公開される前に、より広範な本番環境でのテストを2週間実施し、不具合を発見することです。ユーザーは、リリース候補版の機能がリリース日にも同じように動作すると確信できます。
   ただし、重大なバグが見つかった場合は、明確な説明を付けて、根本的な動作を変更または削除する権利を留保します。
- **リリース済み:** 本番環境ですぐに使用できます。
- **試験的:** 一般公開用にリリースする機能で、現状のままでも使用可能であると考えていますが、追加の注意事項を文書化する場合があります。
- **未文書化:** これらは、<Constant name="core" /> 機能のサブセットであり、内部的に使用されており、契約に含まれておらず、意図的に文書化されていません。この機能は、当該リリースの製品領域の一部とみなさないでください。
- **非推奨:** この状態の機能は、dbt Labs によって積極的に開発または拡張されておらず、削除日まで現状のまま機能します。
- **削除済み:** 削除された機能は、いかなるレベルの製品機能またはプラットフォームサポートも提供されなくなりました。

### dbt Fusion engine

The <Constant name="fusion_engine" /> is currently in beta.

- **Beta:** Beta features are still in development and are only available to select customers. Beta features are incomplete and might not be entirely stable; they should be used at the customer’s risk, as breaking changes could occur. Beta features might not be fully documented, technical support is limited, and service level objectives (SLOs) might not be provided. Download the [Beta Features Terms and Conditions](/assets/beta-tc.pdf) for more details.

- **Path to Generally available (GA):** Learn what's required for the dbt Fusion engine to reach GA in our [Path to GA](/blog/dbt-fusion-engine-path-to-ga) blog post.

