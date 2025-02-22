---
title: ディザスター リカバリーとストレージ アカウントのフェールオーバー (プレビュー) - Azure Storage
description: Azure Storage では、geo 冗長ストレージ アカウントのアカウント フェールオーバー (プレビュー) がサポートされています。 アカウントのフェールオーバーでは、プライマリ エンドポイントが使用できなくなった場合に、ストレージ アカウントのフェールオーバー プロセスを開始できます。
services: storage
author: tamram
ms.service: storage
ms.topic: article
ms.date: 02/25/2019
ms.author: tamram
ms.reviewer: cbrooks
ms.subservice: common
ms.openlocfilehash: b2cd7232bce674dfa5aa2c6f4b6d9386fa7a189b
ms.sourcegitcommit: aebe5a10fa828733bbfb95296d400f4bc579533c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/05/2019
ms.locfileid: "70376445"
---
# <a name="disaster-recovery-and-storage-account-failover-preview-in-azure-storage"></a>Azure Storage でのディザスター リカバリーとストレージ アカウントのフェールオーバー (プレビュー)

Microsoft は、Azure サービスを常に使用できるようにする作業に取り組んでいます。 そうはいっても、計画されていないサービスの停止が発生する可能性はあります。 アプリケーションで回復性が必要な場合は、geo 冗長ストレージを使用して、データが 2 番目のリージョンにレプリケートされるようにすることをお勧めします。 さらに、お客様は、リージョン規模のサービス停止に対処するため、ディザスター リカバリー計画を用意する必要があります。 ディザスター リカバリー計画の重要な部分は、プライマリ エンドポイントが使用できなくなった場合に、セカンダリ エンドポイントにフェールオーバーするための準備です。 

Azure Storage では、geo 冗長ストレージ アカウントのアカウント フェールオーバー (プレビュー) がサポートされています。 アカウントのフェールオーバーでは、プライマリ エンドポイントが使用できなくなった場合に、ストレージ アカウントのフェールオーバー プロセスを開始できます。 フェールオーバーでは、セカンダリ エンドポイントが更新されて、ストレージ アカウントのプライマリ エンドポイントになります。 フェールオーバーが完了すると、クライアントは新しいプライマリ エンドポイントへの書き込みを開始できます。

この記事では、アカウントのフェールオーバーに関する概念とプロセスについて、および顧客への影響が最小限になるようにストレージ アカウントの復旧を準備する方法について説明します。 Azure portal または PowerShell でアカウントのフェールオーバーを開始する方法については、「[Initiate an account failover (preview) (アカウントのフェールオーバー (プレビュー) を開始する)](storage-initiate-account-failover.md)」をご覧ください。


[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="choose-the-right-redundancy-option"></a>適切な冗長性オプションを選択する

すべてのストレージ アカウントは冗長性のためにレプリケートされます。 アカウントに対して選択する冗長性オプションは、必要な回復性の程度によって異なります。 リージョン規模の障害に対する保護の場合は、geo 冗長ストレージを選択し、セカンダリ リージョンからの読み取りアクセスのオプションは指定しても指定しなくてもかまいません。  

**geo 冗長ストレージ (GRS)** では、少なくとも何百キロも離れた 2 つの地理的リージョンの間で非同期的にデータがレプリケートされます。 プライマリ リージョンで障害が発生した場合、セカンダリ リージョンがデータの冗長ソースとして機能します。 ユーザーがフェールオーバーを開始して、セカンダリ エンドポイントをプライマリ エンドポイントに変換することができます。

**読み取りアクセス geo 冗長ストレージ (RA-GRS)** では、セカンダリ エンドポイントへの読み取りアクセスの追加ベネフィットがある geo 冗長ストレージが提供されます。 プライマリ エンドポイントで障害が発生した場合、RA-GRS 用に構成され、高可用性対応に設計されているアプリケーションでは、セカンダリ エンドポイントから引き続き読み取ることができます。 アプリケーションの最大限の回復性のためには RA-GRS をお勧めします。

Azure Storage の他の冗長性オプションとしては、1 つのリージョン内の可用性ゾーン間でデータがレプリケートされるゾーン冗長ストレージ (ZRS)、および 1 つのリージョン内の 1 つのデータ センター内でデータがレプリケートされるローカル冗長ストレージ (LRS) があります。 ストレージ アカウントが ZRS または LRS 用に構成されている場合、GRS または RA-GRS を使用するようにアカウントを変換できます。 geo 冗長ストレージ用にアカウントを構成すると、追加のコストが発生します。 詳細については、「[Azure Storage のレプリケーション](storage-redundancy.md)」をご覧ください。

> [!NOTE]
> geo ゾーン冗長ストレージ (GZRS) および読み取りアクセス geo ゾーン冗長ストレージ (RA-GZRS) は現在プレビュー段階ですが、お客様が管理するアカウントのフェールオーバーと同じリージョンではまだ利用できません。 このため、お客様は現在、GZRS および RA-GZRS アカウントでアカウント フェールオーバー イベントを管理できません。 プレビュー期間中、GZRS/RA-GZRS アカウントに影響するフェールオーバー イベントはすべて Microsoft によって管理されます。

> [!WARNING]
> geo 冗長ストレージでは、データ損失のリスクが伴います。 データはセカンダリ リージョンに非同期的にレプリケートされるため、データがプライマリ リージョンに書き込まれてからセカンダリ リージョンに書き込まれるまでの間に遅延があります。 障害が発生した時点で、セカンダリ エンドポイントにまだレプリケートされていないプライマリ エンドポイントへの書き込み操作は失われます。

## <a name="design-for-high-availability"></a>高可用性向けの設計

最初から高可用性対応にアプリケーションを設計することが重要です。 アプリケーションの設計およびディザスター リカバリーの計画に関するガイダンスについては、次の Azure リソースを参照してください。

* [回復性に優れた Azure 用アプリケーションの設計](https://docs.microsoft.com/azure/architecture/resiliency/):Azure で高可用性アプリケーションを設計するための主要な概念の概要です。
* [可用性のチェックリスト](https://docs.microsoft.com/azure/architecture/checklist/availability):アプリケーションで高可用性向け設計のベスト プラクティスが実装されていることを確認するためのチェックリストです。
* [RA-GRS を使用した高可用性アプリケーションの設計](storage-designing-ha-apps-with-ragrs.md):RA-GRS を利用するアプリケーションを構築するための設計ガイダンスです。
* [チュートリアル:Blob Storage を使用して高可用性アプリケーションを作成する](../blobs/storage-create-geo-redundant-storage.md):障害および復旧がシミュレートされたらエンドポイント間で自動的に切り替わる高可用性アプリケーションを構築する方法を示すチュートリアルです。 

さらに、Azure Storage のデータの高可用性を維持するためには、次のベスト プラクティスに留意してください。

* **ディスク:** [Azure Backup](https://azure.microsoft.com/services/backup/) を使用して、Azure 仮想マシンで使用される VM ディスクをバックアップします。 また、[Azure Site Recovery](https://azure.microsoft.com/services/site-recovery/) を使用して地域的な災害が発生した場合の VM の保護も検討します。
* **ブロック BLOB:** [ソフト削除](../blobs/storage-blob-soft-delete.md)を有効にしてオブジェクトレベルの削除および上書きから保護するか、[AzCopy](storage-use-azcopy.md)、[Azure PowerShell](storage-powershell-guide-full.md)、または [Azure Data Movement Library](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/) を使用して、他のリージョンの別のストレージ アカウントにブロック BLOB をコピーします。
* **ファイル:** [AzCopy](storage-use-azcopy.md) または [Azure PowerShell](storage-powershell-guide-full.md) を使用して、他のリージョンの別のストレージ アカウントにファイルをコピーします。
* **テーブル:** [AzCopy](storage-use-azcopy.md) を使用して、テーブル データを、他のリージョンの別のストレージ アカウントにエクスポートします。

## <a name="track-outages"></a>障害を追跡する

お客様は、[Azure Service Health ダッシュボード](https://azure.microsoft.com/status/)をサブスクライブして、Azure Storage と他の Azure サービスの正常性と状態を追跡できます。

また、書き込みエラーの可能性に対処するようアプリケーションを設計することもお勧めします。 アプリケーションでは、プライマリ リージョンでの障害の可能性があることを通知するように書き込みエラーを公開する必要があります。

## <a name="understand-the-account-failover-process"></a>アカウントのフェールオーバー プロセスを理解する

お客様が管理するアカウントのフェールオーバー (プレビュー) では、何らかの理由でプライマリが使用できなくなった場合、ストレージ アカウント全体をセカンダリ リージョンにフェールオーバーすることができます。 セカンダリ リージョンへのフェールオーバーを強制的に実行すると、クライアントは、フェールオーバー完了後にセカンダリ エンドポイントへのデータの書き込みを開始することができます。 フェールオーバーには、通常、約 1 時間かかります。

### <a name="how-an-account-failover-works"></a>アカウントのフェールオーバーのしくみ

通常の状況では、クライアントのデータはプライマリ リージョンの Azure ストレージ アカウントに書き込まれ、そのデータはセカンダリ リージョンに非同期的にレプリケートされます。 次の図では、プライマリ リージョンが使用可能な場合のシナリオを示します。

![クライアントはプライマリ リージョンのストレージ アカウントにデータを書き込む](media/storage-disaster-recovery-guidance/primary-available.png)

プライマリ エンドポイントが何らかの使用不能になると、クライアントはストレージ アカウントに書き込むことができなくなります。 次の図は、プライマリが使用できなくなり復旧がまだ行われていないシナリオを示しています。

![プライマリを使用できないため、クライアントはデータを書き込むことができない](media/storage-disaster-recovery-guidance/primary-unavailable-before-failover.png)

お客様は、セカンダリ エンドポイントへのアカウントのフェールオーバーを開始します。 フェールオーバー プロセスでは、Azure Storage によって提供される DNS エントリが更新されて、次の図のように、セカンダリ エンドポイントがストレージ アカウントに対する新しいプライマリ エンドポイントになります。

![お客様がセカンダリ エンドポイントへのアカウントのフェールオーバーを開始する](media/storage-disaster-recovery-guidance/failover-to-secondary.png)

GRS および RA-GRS アカウントの場合は、DNS エントリが更新されて、要求が新しいプライマリ エンドポイントに送られるようになると、書き込みアクセスが復元されます。 BLOB、テーブル、キュー、およびファイルに対する既存のストレージ サービス エンドポイントは、フェールオーバー後も同じまま残ります。

> [!IMPORTANT]
> フェールオーバーが完了すると、ストレージ アカウントは、新しいプライマリ エンドポイントでローカル冗長に構成されます。 新しいセカンダリへのレプリケーションを再開するには、geo 冗長ストレージを再び使用するようにアカウントを構成します (RA-GRS または GRS)。
>
> LRS アカウントを RA-GRS または GRS に変換すると費用が発生することに留意してください。 このコストは、フェールオーバー後に新しいプライマリ リージョンのストレージ アカウントで RA-GRS または GRS を使用するための更新に適用されます。  

### <a name="anticipate-data-loss"></a>データ損失の可能性

> [!CAUTION]
> アカウントのフェールオーバーでは、通常、ある程度のデータ損失が発生します。 アカウントのフェールオーバーを開始した場合の影響を理解しておく必要があります。  

データはプライマリ リージョンからセカンダリ リージョンに非同期に書き込まれるため、プライマリ リージョンへの書き込みがセカンダリ リージョンにレプリケートされるまでの間に常に遅延があります。 プライマリ リージョンが使用できなくなった場合、最新の書き込みがまだセカンダリ リージョンにレプリケートされていない可能性があります。

強制的にフェールオーバーを行うと、セカンダリ リージョンが新しいプライマリ リージョンになり、ストレージ アカウントがローカル冗長に構成されるため、プライマリ リージョンのデータはすべて失われます。 フェールオーバーが発生した時点でセカンダリに既にレプリケートされているデータはすべて維持されます。 ただし、プライマリに書き込まれたデータのうち、セカンダリにレプリケートされていなかったものは、永久に失われます。 

**[最終同期時刻]** プロパティは、プライマリ リージョンのデータがセカンダリ リージョンに書き込まれたことが保証される最後の時刻を示します。 最終同期時刻より前に書き込まれたすべてのデータはセカンダリで使用できますが、最終同期時刻より後に書き込まれたデータは、セカンダリに書き込まれていない場合があり、失われる可能性があります。 障害が発生した場合はこのプロパティを使用して、アカウントのフェールオーバーを開始することによって可能性があるデータ損失の量を推定します。 

ベスト プラクティスとしては、最終同期時刻を使用して予想されるデータ損失を評価できるようにアプリケーションを設計します。 たとえば、すべての書き込み操作をログに記録している場合は、最後の書き込み操作の時刻を最終同期時刻と比較することで、セカンダリに同期されていない書き込みを特定できます。

### <a name="use-caution-when-failing-back-to-the-original-primary"></a>元のプライマリにフェールバックするときは注意が必要である

プライマリ リージョンからセカンダリ リージョンにフェールオーバーした後、ストレージ アカウントは新しいプライマリ リージョンでローカル冗長に構成されます。 GRS または RA-GRS を使用するように更新することで、アカウントを再び geo 冗長に構成できます。 フェールオーバー後にアカウントが再び geo 冗長に構成されると、新しいプライマリ リージョンは直ちに新しいセカンダリ リージョン (元のフェールオーバーの前にプライマリであったもの) へのデータのレプリケートを開始します。 ただし、プライマリの既存データが完全に新しいセカンダリにレプリケートされるまで、しばらくかかる場合があります。

ストレージ アカウントが geo 冗長に再構成された後、新しいプライマリから新しいセカンダリへの別のフェールオーバーが開始される可能性があります。 この場合、フェールオーバー前の元のプライマリ リージョンが再びプライマリ リージョンになり、ローカル冗長に構成されます。 その場合、フェールオーバー後のプライマリ リージョン (元のセカンダリ) のすべてのデータが失われます。 フェールバックの前にストレージ アカウントのほとんどのデータが新しいセカンダリにレプリケートされていなかった場合、大きなデータ損失が発生する可能性があります。 

大きなデータ損失を防ぐため、フェールバックを行う前に、 **[最終同期時刻]** プロパティの値を確認してください。 最終同期時刻を、新しいプライマリにデータが書き込まれた最後の時刻と比較して、予想されるデータ損失を評価します。 

## <a name="initiate-an-account-failover"></a>アカウントのフェールオーバーを開始する

アカウントのフェールオーバーは、Azure portal、PowerShell、Azure CLI、または Azure Storage リソース プロバイダー API から開始できます。 フェールオーバーを開始する方法について詳しくは、「[Initiate an account failover (preview) (アカウントのフェールオーバー (プレビュー) を開始する)](storage-initiate-account-failover.md)」をご覧ください。

## <a name="about-the-preview"></a>プレビューについて

アカウントのフェールオーバーは、Azure Resource Manager デプロイで GRS または RA-GRS を使用するすべてのお客様がプレビューで使用できます。 汎用 v1、汎用 v2、および BLOB のストレージ アカウントの種類がサポートされています。 現在、アカウントのフェールオーバーは次のリージョンで利用できます。

- 米国西部 2
- 米国中西部

プレビューは、非運用環境のみでの使用を意図されています。 運用環境のサービス レベル契約(SLA) は現在使用できません。

### <a name="register-for-the-preview"></a>プレビューに登録する

プレビューに登録するには、PowerShell から次のコマンドを実行します。 忘れずに、角かっこ内のプレースホルダーを自分のサブスクリプション ID に置き換えてください。

```powershell
Connect-AzAccount -SubscriptionId <subscription-id>
Register-AzProviderFeature -FeatureName CustomerControlledFailover -ProviderNamespace Microsoft.Storage
```

プレビューの承認を受け取るまで、1 ～ 2 日かかる場合があります。 登録が承認されたことを確認するには、次のコマンドを実行します。

```powershell
Get-AzProviderFeature -FeatureName CustomerControlledFailover -ProviderNamespace Microsoft.Storage
```

### <a name="additional-considerations"></a>追加の考慮事項 

このセクションで説明されている追加の考慮事項を確認し、プレビュー期間中にフェールオーバーを強制した場合にアプリケーションとサービスが受ける可能性がある影響について理解してください。

#### <a name="azure-virtual-machines"></a>Azure Virtual Machines

Azure 仮想マシン (VM) は、アカウントのフェールオーバーの一部としてフェイルオーバーされません。 プライマリ リージョンが使用不能になり、セカンダリ リージョンにフェールオーバーする場合は、フェールオーバー後に VM を再作成する必要があります。 

#### <a name="azure-unmanaged-disks"></a>Azure アンマネージド ディスク

ベスト プラクティスとして、アンマネージド ディスクをマネージド ディスクに変換することをお勧めします。 ただし、Azure VM にアタッチされているアンマネージド ディスクを含むアカウントをフェールオーバーする必要がある場合は、フェールオーバーを始める前に、VM をシャットダウンする必要があります。

アンマネージド ディスクは、Azure Storage にページ BLOB として格納されます。 VM が Azure で実行されていると、VM にアタッチされているアンマネージド　ディスクはリースされます。 BLOB にリースがある場合、アカウントのフェールオーバーは続行できません。 フェールオーバーを実行するには、次の手順のようにします。

1. 始める前に、アンマネージド ディスクの名前、その論理ユニット番号 (LUN)、ディスクがアタッチされている VM を記録しておきます。 そうすることで、フェールオーバー後にディスクを再アタッチするのが容易になります。 
2. VM をシャット ダウンします。
3. VM を削除します。ただし、アンマネージド ディスクの VHD ファイルは残しておきます。 VM を削除した時刻を記録しておきます。
4. **[最終同期時刻]** が更新されて、VM を削除した時刻より後になるまで待ちます。 フェールオーバーが発生した時点で、セカンダリ エンドポイントの VHD ファイルが完全に更新されていない場合、新しいプライマリ リージョンで VM が正常に機能しない可能性があるので、このステップは重要です。
5. アカウントのフェールオーバーを開始します。
6. アカウントのフェールオーバーが完了し、セカンダリ リージョンが新しいプライマリ リージョンになるまで待機します。
7. 新しいプライマリ リージョンで VM を作成し、VHD を再アタッチします。
8. 新しい VM を起動します。

VM をシャットダウンすると、一時ディスクに格納されているすべてのデータが失われることに留意してください。

### <a name="unsupported-features-or-services"></a>サポートされていない機能またはサービス
次の機能またはサービスは、プレビュー リリースのアカウントのフェールオーバーではサポートされていません。

- Azure File Sync では、ストレージ アカウントのフェールオーバーはサポートされていません。 Azure File Sync でクラウド エンドポイントとして使用されている Azure ファイル共有を含むストレージ アカウントは、フェールオーバーしないでください。 それを行うと、同期の動作が停止し、新しく階層化されたファイルの場合は予期せずデータが失われる可能性があります。  
- アーカイブ済みの BLOB 含むストレージ アカウントは、フェールオーバーできません。 アーカイブ済み BLOB は、フェールオーバーする計画がない別のストレージ アカウントで保持してください。
- Premium ブロック BLOB 含むストレージ アカウントは、フェールオーバーできません。 現在、Premium ブロック BLOB をサポートするストレージ アカウントでは、geo 冗長がサポートされていません。
- フェールオーバーの完了後、フェールオーバー元で有効になっている場合は、[イベント サブスクリプション](https://docs.microsoft.com/azure/storage/blobs/storage-blob-event-overview)、[ライフサイクル ポリシー](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts)、[Storage Analytics Logging](https://docs.microsoft.com/rest/api/storageservices/about-storage-analytics-logging) の動作が停止します。

## <a name="copying-data-as-an-alternative-to-failover"></a>フェールオーバーの代わりとしてのデータのコピー

ストレージ アカウントが RA-GRS 用に構成されている場合、セカンダリ エンドポイントを使用するデータへの読み取りアクセス権があります。 プライマリ リージョンで障害が発生したときにフェールオーバーしたくない場合は、[AzCopy](storage-use-azcopy.md)、[Azure PowerShell](storage-powershell-guide-full.md)、[Azure Data Movement Library](https://azure.microsoft.com/blog/introducing-azure-storage-data-movement-library-preview-2/) などのツールを使用して、セカンダリ リージョンのストレージ アカウントから、影響を受けていないリージョンの別のストレージ アカウントに、データをコピーできます。 その後は、アプリケーションの参照先をそのストレージ アカウントにして、読み取りと書き込みの両方に利用できます。

## <a name="microsoft-managed-failover"></a>Microsoft が管理するフェールオーバー

大きな災害のためにリージョンが失われるような極端な状況では、Microsoft がリージョン間のフェールオーバーを開始できます。 この場合、ユーザーによる操作は必要ありません。 Microsoft 管理のフェールオーバーが完了するまで、ストレージ アカウントに書き込みアクセスを行うことはできません。 ストレージ アカウントが RA-GRS 対応に構成されている場合は、アプリケーションはセカンダリ リージョンから読み取ることができます。 

## <a name="see-also"></a>関連項目

* [アカウントのフェールオーバー (プレビュー) を開始する](storage-initiate-account-failover.md)
* [RA-GRS を使用した高可用性アプリケーションの設計](storage-designing-ha-apps-with-ragrs.md)
* [チュートリアル:Blob Storage を使用して高可用性アプリケーションを作成する](../blobs/storage-create-geo-redundant-storage.md) 
