---
title: Azure API Management のセキュリティ コントロール
description: API Management を評価するためのセキュリティ コントロールのチェックリスト
services: api-management
author: msmbaldwin
manager: rkarlin
ms.service: api-management
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: mbaldwin
ms.openlocfilehash: e808f373ed3c977fb3263bc9e2e25bc602c7a7e1
ms.sourcegitcommit: 7c5a2a3068e5330b77f3c6738d6de1e03d3c3b7d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "70886295"
---
# <a name="security-controls-for-api-management"></a>API Management のセキュリティ コントロール

この記事では、API Management に組み込まれているセキュリティ コントロールについて説明します。

[!INCLUDE [Security controls Header](../../includes/security-controls-header.md)]

## <a name="network"></a>ネットワーク

| セキュリティ コントロール | はい/いいえ | メモ |
|---|---|--|
| サービス エンドポイントのサポート| いいえ | |
| VNet インジェクションのサポート| はい | |
| ネットワークの分離とファイアウォールのサポート| はい | それぞれ、ネットワーク セキュリティ グループ (NSG) と Azure Application Gateway (またはその他のソフトウェア アプライアンス) を使用します。 |
| 強制トンネリングのサポート| はい | Azure ネットワークでは、強制トンネリングを提供します。 |

## <a name="monitoring--logging"></a>監視およびログ記録

| セキュリティ コントロール | はい/いいえ | メモ|
|---|---|--|
| Azure 監視サポート (Log analytics や App Insights など)| はい | |
| コントロールと管理プレーンのログ記録と監査| はい | [Azure Monitor のアクティビティ ログ](../azure-monitor/platform/activity-logs-overview.md) |
| データ プレーンのログ記録と監査| はい | [Azure Monitor 診断ログ](../azure-monitor/platform/diagnostic-logs-overview.md)と (必要に応じて) [Azure Application Insights](../azure-monitor/app/app-insights-overview.md)。  |

## <a name="identity"></a>ID

| セキュリティ コントロール | はい/いいえ | メモ|
|---|---|--|
| 認証| はい | |
| Authorization| はい | |

## <a name="data-protection"></a>データ保護

| セキュリティ コントロール | はい/いいえ | メモ |
|---|---|--|
| 保存時のサーバー側の暗号化:Microsoft のマネージド キー | はい | 証明書、キー、およびシークレットという名前付きの値などの機密データは、サービスで管理される、サービス インスタンスごとのキーで暗号化されます。 |
| 保存時のサーバー側の暗号化: カスタマー マネージド キー (BYOK) | いいえ | すべての暗号化キーはサービス インスタンスごとに存在し、サービスで管理されます。 |
| 列レベルの暗号化 (Azure Data Services)| 該当なし | |
| 転送中の暗号化 (ExpressRoute 暗号化、VNet 内の暗号化、および VNet 間暗号化など)| はい | [Express Route](../expressroute/index.yml) と VNet 暗号化は、[Azure ネットワーク](../virtual-network/index.yml)によって提供されます。 |
| API 呼び出しの暗号化| はい | 管理プレーン呼び出しは、TLS 経由で [Azure Resource Manager](../azure-resource-manager/index.yml) によって行われます。 有効な JSON Web トークン (JWT) が必要です。  データ プレーン呼び出しは、TLS と、サポートされる認証メカニズムのいずれか (例: クライアント証明書や JWT など) で保護できます。
 |

## <a name="configuration-management"></a>構成管理

| セキュリティ コントロール | はい/いいえ | メモ|
|---|---|--|
| 構成管理のサポート (構成のバージョン管理など)| はい | [Azure API Management DevOps リソース キット](https://aka.ms/apimdevops)を使用 |

## <a name="vulnerability-scans-false-positives"></a>脆弱性スキャンの偽陽性

このセクションでは、Azure API Management に影響しない一般的な脆弱性について説明します。

| 脆弱性               | 説明                                                                                                                                                                                                                                                                                                               |
|-----------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Ticketbleed (CVE-2016-9244) | Ticketbleed は、一部の F5 製品で検出された TLS SessionTicket 拡張機能の実装に含まれる脆弱性です。 初期化されていないメモリから最大 31 バイトのデータがリーク ("流出") する可能性があります。 これは、クライアントから渡される TLS スタックをセッション ID でパディングして、データを 32 ビットの長さにすることが原因で発生します。 |

## <a name="next-steps"></a>次の手順

- [Azure サービス全体の組み込みセキュリティ コントロール](../security/fundamentals/security-controls.md)について説明します。