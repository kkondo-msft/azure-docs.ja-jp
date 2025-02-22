---
title: サンプル - UK OFFICIAL および UK NHS のブループリント - コントロール マッピング
description: UK OFFICIAL および UK NHS のブループリント サンプルのコントロール マッピング。
services: blueprints
author: DCtheGeek
ms.author: dacoulte
ms.date: 06/26/2019
ms.topic: conceptual
ms.service: blueprints
manager: carmonm
ms.openlocfilehash: 43848e99f679e306747c4cb7b31a4d4692c888cc
ms.sourcegitcommit: 083aa7cc8fc958fc75365462aed542f1b5409623
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "70918337"
---
# <a name="control-mapping-of-the-uk-official-and-uk-nhs-blueprint-samples"></a>UK OFFICIAL および UK NHS のブループリント サンプルのコントロール マッピング

以下の記事では、UK OFFICIAL および UK NHS のブループリント サンプルが、UK OFFICIAL および UK NHS コントロールにどのようにマップされるのかについて詳しく説明します。 コントロールの詳細については、[UK OFFICIAL](https://www.gov.uk/government/publications/government-security-classifications) に関するページをご覧ください。

以下のマッピングでは、マップ先は **UK OFFICIAL** コントロールと **UK NHS** コントロールです。 右側のナビゲーションを使用すると、特定のコントロール マッピングに直接ジャンプできます。 マップ コントロールの多くは、[Azure Policy](../../../policy/overview.md) イニシアチブを使用して実装されますす。 イニシアチブの詳細を確認するには、Azure portal で **[ポリシー]** を開き、 **[定義]** ページを選択します。 次に、 **[\[プレビュー\]: UK OFFICIAL コントロールと UK NHS コントールを監査し、特定の VM 拡張機能をデプロイして監査要件をサポートする]** ビルトイン ポリシー イニシアチブを見つけて選択します。

> [!IMPORTANT]
> 以下の各コントロールは、1 つ以上の [Azure Policy](../../../policy/overview.md) 定義に関連します。 これらのポリシーは、コントロールの[コンプライアンスを評価](../../../policy/how-to/get-compliance-data.md)するのに役立ちますが、多くの場合、コントロールと 1 つ以上のポリシーの間に 1:1 または完全な一致はありません。 そのため、Azure Policy での**準拠**は、ポリシー自体のみを指しています。これによって、コントロールのすべての要件に完全に準拠していることが保証されるわけではありません。 また、コンプライアンス標準には、現時点でどの Azure Policy 定義でも対応されていないコントロールが含まれています。 したがって、Azure Policy でのコンプライアンスは、全体のコンプライアンス状態の部分的ビューでしかありません。 このコンプライアンス ブループリント サンプルのコントロールと Azure Policy 定義の間の関連付けは、時間の経過と共に変わる可能性があります。 変更履歴を表示するには、[GitHub のコミット履歴](https://github.com/MicrosoftDocs/azure-docs/commits/master/articles/governance/blueprints/samples/ukofficial/control-mapping.md)に関するページを参照してください。

## <a name="1-data-in-transit-protection"></a>1 転送中のデータの保護

このブループリントでは、ストレージ アカウントおよび Redis Cache へのセキュリティで保護されていない接続を監査する [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、Azure サービスでの情報転送のセキュリティの確保を支援します。

- Redis Cache に対してセキュリティで保護された接続のみを有効にする必要がある
- ストレージ アカウントへの安全な転送を有効にする必要がある

## <a name="23-data-at-rest-protection"></a>2.3 保存データの保護

このブループリントでは、特定の暗号化コントロールを適用し、脆弱な暗号化設定の使用を監査する [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、暗号化コントロールの使用に関するポリシーの適用を支援します。
最適でない暗号化構成が Azure リソースのどこに存在しているかを把握することにより、適切な是正措置を実施し、リソースの構成を情報セキュリティ ポリシーに準拠させることができます。 具体的には、このブループリントによって割り当てられるポリシーでは、データ レイク ストレージ アカウントの暗号化の要求、SQL データベースでの Transparent Data Encryption の要求、ストレージ アカウント、SQL データベース、仮想マシン ディスク、および自動化アカウント変数での暗号化の不足の監査、ストレージ アカウントおよび Redis Cache へのセキュリティで保護されていない接続の監査、仮想マシンの脆弱なパスワード暗号化の監査、暗号化されていない Service Fabric 通信の監査を行います。

- SQL データベースで Transparent Data Encryption を有効にする必要がある
- 仮想マシンでディスク暗号化を適用する必要がある
- Automation アカウント変数は、暗号化する必要がある
- ストレージ アカウントへの安全な転送を有効にする必要がある
- Service Fabric クラスターでは、ClusterProtectionLevel プロパティを EncryptAndSign に設定する必要がある
- SQL データベースで Transparent Data Encryption を有効にする必要がある
- SQL DB Transparent Data Encryption をデプロイする
- Data Lake Store アカウントの暗号化を要求する
- 許可された場所 ("UK SOUTH" および "UK WEST" にハードコーディングされている)
- リソース グループの許可された場所 ("UK SOUTH" および "UK WEST" にハードコーディングされている)

## <a name="52-vulnerability-management"></a>5.2 脆弱性の管理

このブループリントでは、エンドポイント保護の不足、システムの更新プログラムの不足、オペレーティング システムの脆弱性、SQL の脆弱性、および仮想マシンの脆弱性を監視する [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、情報システムの脆弱性の管理を支援します。 これらの分析情報には、デプロイされたリソースのセキュリティ状態に関するリアルタイムな情報が含まれます。これらの情報は、修復アクションの優先度を決定するのに役立ちます。

- Endpoint Protection の欠落の Azure Security Center での監視
- システム更新プログラムをマシンにインストールする必要がある
- 使用しているマシンでセキュリティ構成の脆弱性を修復する必要がある
- SQL データベースの脆弱性を修復する必要がある
- 脆弱性評価ソリューションによって脆弱性を修復する必要がある

## <a name="53-protective-monitoring"></a>5.3 保護的監視

このブループリントでは、制限のないアクセス、ホワイトリスト アクティビティ、および脅威の保護的監視を行う [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、情報システム資産の保護を支援します。

- ストレージ アカウントに対する制限のないネットワーク アクセスの監査
- 適応型アプリケーション制御を仮想マシンで有効にする必要がある
- SQL サーバーでの脅威検出のデプロイ
- 既定の Windows Server 用 Microsoft IaaS マルウェア対策拡張機能のデプロイ

## <a name="9-secure-user-management--10-identity-and-authentication"></a>9 ユーザー管理のセキュリティ保護/ 10 ID と認証

Azure では、Azure のリソースにアクセスできるユーザーの管理を支援するために、ロールベースのアクセス制御 (RBAC) が実装されています。 Azure リソースにできるユーザーとそのアクセス許可は、Azure portal を使用して確認できます。 このブループリントでは、所有者アクセス許可や読み取り/書き込みアクセス許可を持つ外部アカウントと、所有者アクセス許可や読み取り/書き込みアクセス許可を持ち、多要素認証が有効になっていないアカウントを監査する [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、アクセス権の制限と制御を支援します。

- サブスクリプションで所有者アクセス許可を持つアカウントに対して MFA を有効にする必要がある
- サブスクリプションに対する書き込みアクセス許可を持つアカウントに対して MFA を有効にする必要がある
- サブスクリプションに対する読み取りアクセス許可を持つアカウントに対して MFA を有効にする必要がある
- 所有者アクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある
- 書き込みアクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある
- 読み取りアクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある

このブループリントでは、SQL サーバーと Service Fabric に対する Azure Active Directory 認証の使用を監査する Azure Policy 定義が割り当てられます。 Azure Active Directory 認証を使用すると、アクセス許可の管理を簡単にし、データベース ユーザーとその他の Microsoft サービスの ID を一元管理できます。

- SQL サーバーに対して Azure Active Directory 管理者をプロビジョニングする必要がある
- Service Fabric クラスターは、クライアント認証に Azure Active Directory だけを使用する必要がある

このブループリントでは、優先的にレビューする必要があるアカウント (非推奨アカウントや外部アカウントなど) を監査する Azure Policy 定義も割り当てられます。 必要な場合は、サインインしようとしているアカウントをブロック (または削除) して、Azure リソースへのアクセス権を直ちに削除することもできます。 このブループリントでは、削除を検討する必要がある非推奨アカウントを監査する 2 つの Azure Policy 定義が割り当てられます。

- 非推奨のアカウントをサブスクリプションから削除する必要がある
- 所有者としてのアクセス許可を持つ非推奨のアカウントをサブスクリプションから削除する必要がある
- 所有者アクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある
- 書き込みアクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある

このブループリントでは、Linux VM のパスワード ファイルのアクセス許可を監査して、それらが正しく設定されていない場合にアラートを生成する Azure Policy 定義も割り当てられます。 この設計により、認証子が侵害されないように是正措置を講じることができます。

- \[プレビュー\]:Linux VM /etc/passwd ファイルのアクセス許可が 0644 に設定されていることを監査する

このブループリントでは、最低限の強度や他のパスワード要件が適用されていない Windows VM を監査する Azure Policy 定義を割り当てることで、強力なパスワード の適用を支援します。 パスワード強度のポリシーに違反している VM を把握できるようになるので、適切な是正措置を実施し、すべての VM ユーザー アカウントに対して、パスワード ポリシーへの準拠を徹底させることができます。

- \[プレビュー\]:パスワードの複雑さの設定が有効になっていない Windows VM を監査する要件をデプロイする
- \[プレビュー\]:パスワードの有効期間が 70 日になっていない Windows VM を監査する要件をデプロイする
- \[プレビュー\]:パスワードの変更禁止期間が 1 日になっていない Windows VM を監査する要件をデプロイする
- \[プレビュー\]:パスワードの最小文字数が 14 文字に制限されていない Windows VM を監査する要件をデプロイする
- \[プレビュー\]:以前の 24 個のパスワードの再利用が許可されている Windows VM を監査する要件をデプロイする
- \[プレビュー\]:パスワードの複雑さの設定が有効になっていない Windows VM を監査する
- \[プレビュー\]:パスワードの有効期間が 70 日になっていない Windows VM を監査する
- \[プレビュー\]:パスワードの変更禁止期間が 1 日になっていない Windows VM を監査する
- \[プレビュー\]:パスワードの最小文字数が 14 文字に制限されていない Windows VM を監査する
- \[プレビュー\]:以前の 24 個のパスワードの再利用が許可されている Windows VM を監査する

このブループリントでは、Azure Policy 定義を割り当てることで、Azure リソースへのアクセスを制御することもできます。 これらのポリシーは、リソースの種類や構成の使用状況を監査することで、リソースへのアクセスをより厳格に制限するためのものです。 これらのポリシーに違反しているリソースを把握することで、適切な是正措置を実施し、承認済みのユーザーだけが Azure リソースにアクセスできるようにすることができます。

- \[プレビュー\]:パスワードなしのアカウントが存在する Linux VM を監査する要件をデプロイする
- \[プレビュー\]:パスワードなしのアカウントからのリモート接続が許可されている Linux VM を監査する要件をデプロイする
- \[プレビュー\]:パスワードなしのアカウントが存在する Linux VM を監査する
- \[プレビュー\]:パスワードなしのアカウントからのリモート接続が許可されている Linux VM を監査する
- ストレージ アカウントを新しい Azure Resource Manager リソースに移行する必要がある
- 仮想マシンを新しい Azure Resource Manager リソースに移行する必要がある
- マネージド ディスクを使用していない VM の監査

## <a name="11-external-interface-protection"></a>11 外部インターフェイスの保護

このブループリントでは、セキュリティで保護された適切なユーザー管理を実現するために 25 個以上のポリシーを使用する以外に、制限のないストレージ アカウントを監視する [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、サービス インターフェイスを未承認のアクセスから保護できるよう支援します。 アクセス制限のないストレージ アカウントにより、情報システム内の情報に対する意図しないアクセスが許可される可能性があります。 このブループリントでは、仮想マシンで適応型アプリケーション制御を有効にするポリシーも割り当てられます。

- ストレージ アカウントに対する制限のないネットワーク アクセスの監査
- 適応型アプリケーション制御を仮想マシンで有効にする必要がある

## <a name="12-secure-service-administration"></a>12 サービス管理のセキュリティ保護

Azure では、Azure のリソースにアクセスできるユーザーの管理を支援するために、ロールベースのアクセス制御 (RBAC) が実装されています。 Azure リソースにできるユーザーとそのアクセス許可は、Azure portal を使用して確認できます。 このブループリントでは、所有者アクセス許可や書き込みアクセス許可を持つ外部アカウントと、所有者アクセス許可や書き込みアクセス許可を持ち、多要素認証が有効になっていないアカウントを監査する 5 つの [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、特権アクセス権の制限と制御を支援します。

クラウド サービスの管理に使用されるシステムには、そのサービスへの特権アクセスが認められます。 このシステムがセキュリティ侵害されると、セキュリティ制御をすり抜けて、大量のデータの漏洩や不正操作を含む重大な影響がおよびます。 サービス プロバイダーの管理者が運用サービスの管理に使用する方法は、サービスのセキュリティを低下させる可能性がある悪用のリスクを軽減するように設計する必要があります。 この原則が実施されていない場合、攻撃者はセキュリティ制御をすり抜け、大量のデータを窃取または操作する手段を得るおそれがあります。

- サブスクリプションで所有者アクセス許可を持つアカウントに対して MFA を有効にする必要がある
- サブスクリプションに対する書き込みアクセス許可を持つアカウントに対して MFA を有効にする必要がある
- 所有者アクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある
- 書き込みアクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある

このブループリントでは、SQL サーバーと Service Fabric に対する Azure Active Directory 認証の使用を監査する Azure Policy 定義が割り当てられます。 Azure Active Directory 認証を使用すると、アクセス許可の管理を簡単にし、データベース ユーザーとその他の Microsoft サービスの ID を一元管理できます。

- SQL サーバーに対して Azure Active Directory 管理者をプロビジョニングする必要がある
- Service Fabric クラスターは、クライアント認証に Azure Active Directory だけを使用する必要がある

このブループリントでは、優先的にレビューする必要があるアカウント (非推奨アカウントや管理者特権のアクセス許可を持つ外部アカウントなど) を監査する Azure Policy 定義も割り当てられます。 必要な場合は、サインインしようとしているアカウントをブロック (または削除) して、Azure リソースへのアクセス権を直ちに削除することもできます。 このブループリントでは、削除を検討する必要がある非推奨アカウントを監査する 2 つの Azure Policy 定義が割り当てられます。

- 非推奨のアカウントをサブスクリプションから削除する必要がある
- 所有者としてのアクセス許可を持つ非推奨のアカウントをサブスクリプションから削除する必要がある
- 所有者アクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある
- 書き込みアクセス許可を持つ外部アカウントをサブスクリプションから削除する必要がある

このブループリントでは、Linux VM のパスワード ファイルのアクセス許可を監査して、それらが正しく設定されていない場合にアラートを生成する Azure Policy 定義も割り当てられます。 この設計により、認証子が侵害されないように是正措置を講じることができます。

- \[プレビュー\]:Linux VM /etc/passwd ファイルのアクセス許可が 0644 に設定されていることを監査する

## <a name="13-audit-information-for-users"></a>13 ユーザー監査情報

このブループリントでは、Azure リソースのログ設定を監査する [Azure Policy](../../../policy/overview.md) 定義を割り当てることで、システム イベントのログ記録の徹底を支援します。 割り当てられたポリシーでは、指定された Log Analytics ワークスペースにログを送信していない仮想マシンがないかどうかも監査されます。

- SQL Server の高度なデータ セキュリティ設定で監査を有効にする必要がある
- 診断設定の監査
- SQL サーバー レベルの監査設定の監査
- \[プレビュー\]:Linux VM への Log Analytics エージェントのデプロイ
- \[プレビュー\]:Windows VM への Log Analytics エージェントのデプロイ
- 仮想ネットワーク作成時の Network Watcher のデプロイ

## <a name="next-steps"></a>次の手順

UK OFFICIAL および UK NHS のブループリントのコントロール マッピングを確認しました。次に、以下の記事で概要およびこのサンプルをデプロイする方法を確認します。

> [!div class="nextstepaction"]
> [UK OFFICIAL および UK NHS のブループリント - 概要](./index.md)
> [UK OFFICIAL および UK NHS のブループリント - デプロイ手順](./deploy.md)

ブループリントとその使用方法に関するその他の記事:

- [ブループリントのライフサイクル](../../concepts/lifecycle.md)を参照する。
- [静的および動的パラメーター](../../concepts/parameters.md)の使用方法を理解する。
- [ブループリントの優先順位](../../concepts/sequencing-order.md)のカスタマイズを参照する。
- [ブループリントのリソース ロック](../../concepts/resource-locking.md)の使用方法を調べる。
- [既存の割り当ての更新](../../how-to/update-existing-assignments.md)方法を参照する。