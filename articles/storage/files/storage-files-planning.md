---
title: Azure Files のデプロイの計画 | Microsoft Docs
description: Azure Files のデプロイを計画するときの考慮事項について説明します。
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 04/25/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 30842c787e2009b4919fef916f3c5e1f73a79bf2
ms.sourcegitcommit: 083aa7cc8fc958fc75365462aed542f1b5409623
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/11/2019
ms.locfileid: "70918813"
---
# <a name="planning-for-an-azure-files-deployment"></a>Azure Files のデプロイの計画

[Azure Files](storage-files-introduction.md) はクラウドで、業界標準の SMB プロトコルを介してアクセスできる、フル マネージドのファイル共有を提供します。 Azure Files は完全に管理されているため、運用環境へのデプロイは、ファイル サーバーまたは NAS デバイスをデプロイして管理するよりはるかに簡単です。 この記事では、組織内で運用するために Azure ファイル共有をデプロイするときの考慮事項を説明します。

## <a name="management-concepts"></a>管理の概念

 次の図は、Azure Files の管理の構造を示しています。

![ファイル構造](./media/storage-files-introduction/files-concepts.png)

* **[ストレージ アカウント]** : Azure のストレージにアクセスする場合には必ず、ストレージ アカウントを使用します。 ストレージ アカウントの容量の詳細については、[拡張性とパフォーマンスのターゲット](../common/storage-scalability-targets.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)に関するページを参照してください。

* **共有**:File Storage 共有は、Azure 内の SMB ファイル共有です。 ディレクトリとファイルはすべて親の共有に作成する必要があります。 アカウントに含まれる共有の数と、共有に格納できるファイル数には制限がなく、ファイル共有の合計容量まで増やすことができます。 Standard ファイル共有の合計容量は最大 5 TiB (GA) または 100 TiB (プレビュー) です。Pemium ファイル共有の合計容量は最大 100 TiB です。

* **ディレクトリ**:ディレクトリの階層 (オプション)。

* **ファイル**:共有内のファイル。 ファイルのサイズの上限は 1 TiB です。

* **URL の形式**:ファイル REST プロトコルで Azure ファイル共有に対する要求を行う場合、次の URL 形式を使ってファイルをアドレス指定できます。

    ```
    https://<storage account>.file.core.windows.net/<share>/<directory>/<file>
    ```

## <a name="data-access-method"></a>データ アクセスの方法

Azure Files には 2 つの便利なデータ アクセス方法が組み込まれており、単独で、または組み合わせて使って、データにアクセスできます。

1. **クラウドへの直接アクセス**:業界標準のサーバー メッセージ ブロック (SMB) プロトコルまたはファイル REST API を使って、[Windows](storage-how-to-use-files-windows.md)、[macOS](storage-how-to-use-files-mac.md)、[Linux](storage-how-to-use-files-linux.md) で Azure ファイル共有をマウントできます。 SMB では、共有上のファイルの読み取り/書き込みは、Azure のファイル共有に対して直接行われます。 Azure の VM によってマウントするには、OS の SMB クライアントが少なくとも SMB 2.1 をサポートしている必要があります。 ユーザーのワークステーションなどのオンプレミスにマウントするには、ワークステーションでサポートされている SMB クライアントが、少なくとも SMB 3.0 (暗号化付き) をサポートしている必要があります。 SMB に加えて、新しいアプリケーションまたはサービスはファイル REST を使ってファイル共有に直接アクセスできます。ファイル REST は、簡単でスケーラブルなソフトウェア開発用アプリケーション プログラミング インターフェイスを備えています。
2. **Azure File Sync**:Azure File Sync を使用すると、オンプレミスまたは Azure の Windows Server に共有をレプリケートできます。 ユーザーは、SMB や NFS 共有などを使って Windows Server 経由でファイル共有にアクセスします。 この方法は、ブランチ オフィスなどの Azure データ センターから離れた場所にあるデータにアクセスして変更する場合に便利です。 複数のブランチ オフィス間など、複数の Windows Server エンドポイント間でデータをレプリケートできます。 最後に、Azure Files にデータを階層化できます。このようにすると、Server 経由ですべてのデータにアクセスできることは変わりませんが、Server にデータが完全にコピーされることはありません。 ユーザーがファイルを開くと、データはシームレスに回収されます。

次の表では、ユーザーおよびアプリケーションが Azure ファイル共有にアクセスできる方法を示します。

| | クラウドへの直接アクセス | Azure ファイル同期 |
|------------------------|------------|-----------------|
| 使う必要があるプロトコル | Azure Files は、SMB 2.1、SMB 3.0、ファイル REST API をサポートします。 | Windows Server でサポートされているプロトコル (SMB、NFS、FTPS など) を使って、Azure ファイル共有にアクセスします。 |  
| ワークロードを実行する場所 | **Azure 内**:Azure Files によりデータへの直接アクセスが提供されます。 | **低速ネットワークのオンプレミス**:Windows、Linux、macOS のクライアントは、ローカルなオンプレミスの Windows ファイル共有を、Azure ファイル共有の高速キャッシュとしてマウントすることができます。 |
| 必要な ACL のレベル | 共有とファイルのレベル。 | 共有、ファイル、ユーザーのレベル。 |

## <a name="data-security"></a>データのセキュリティ

Azure Files には、データのセキュリティを確保するための複数の組み込みオプションがあります。

* ネットワーク プロトコルでの暗号化のサポート:SMB 3.0 暗号化と HTTPS 経由のファイル REST。 既定での動作は次のとおりです。 
    * SMB 3.0 暗号化をサポートするクライアントは、暗号化されたチャネルでデータを送受信します。
    * 暗号化付き SMB 3.0 をサポートしていないクライアントは、暗号化を行わない SMB 2.1 または SMB 3.0 経由でデータセンター内通信を行うことができます。 SMB クライアントは、暗号化を行わない SMB 2.1 または SMB 3.0 経由でデータセンター間通信を行うことはできません。
    * クライアントは、HTTP または HTTPS を使ってファイル REST 経由で通信できます。
* 保存時の暗号化 ([Azure Storage Service Encryption](../common/storage-service-encryption.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)):Storage Service Encryption (SSE) は、すべてのストレージ アカウントに対して有効になりました。 保存時のデータは完全に管理されたキーで暗号化されます。 保存データの暗号化では、ストレージ コストの増加やパフォーマンスの低下はありません。 
* オプションの転送中のデータの暗号化要件: 選択すると、Azure Files は暗号化されていないチャネル経由でのデータへのアクセスが拒否されます。 具体的には、暗号化接続を使う HTTPS と SMB 3.0 だけが許可されます。

    > [!Important]  
    > データのセキュリティで保護された転送を要求すると、暗号化ありで SMB 3.0 と通信する機能のない古い SMB クライアントは失敗します。 詳しくは、[Windows でのマウント](storage-how-to-use-files-windows.md)、[Linux でのマウント](storage-how-to-use-files-linux.md)、および [macOS でのマウント](storage-how-to-use-files-mac.md)に関するページをご覧ください。

最大限のセキュリティのため、保存時の暗号化と、最新のクライアントを使ってデータにアクセスするときの転送中のデータの暗号化を、両方とも常に有効にすることを強くお勧めします。 たとえば、SMB 2.1 のみをサポートする Windows Server 2008 R2 VM に共有をマウントする必要がある場合は、SMB 2.1 は暗号化をサポートしていないため、ストレージ アカウントへの暗号化されていないトラフィックを許可する必要があります。

Azure ファイル同期を使って Azure ファイル共有にアクセスする場合は、保存時のデータの暗号化が要求されているかどうかにかかわらず、Windows Server へのデータの同期には、暗号化ありの HTTPS と SMB 3.0 が常に使われます。

## <a name="file-share-performance-tiers"></a>ファイル共有のパフォーマンス レベル

Azure Files には、Standard と Premium の 2 つのパフォーマンス レベルが用意されています。

### <a name="standard-file-shares"></a>Standard ファイル共有

Standard ファイル共有は、ハード ディスク ドライブ (HDD) によってサポートされます。 Standard ファイル共有は、汎用のファイル共有や開発/テスト環境などのパフォーマンスの変動の影響を受けにくい IO ワークロードに対して信頼性の高いパフォーマンスを提供します。 Standard ファイル共有は、従量課金制の課金モデルでのみ利用できます。

サイズが最大 5 TiB の Standard ファイル共有は、GA オファリングとして利用できます。 一方、より大きなファイル共有 (5 TiB を超え、最大 100 TiB までの共有) は、現在プレビュー オファリングとして利用できます。

> [!IMPORTANT]
> オンボードの手順と、プレビューの範囲と制限については、「[大きなファイル共有へのオンボード (Standard レベル)](#onboard-to-larger-file-shares-standard-tier)」セクションを参照してください。

### <a name="premium-file-shares"></a>Premium ファイル共有

Premium ファイル共有は、ソリッドステート ドライブ (SSD) によって支えられています。 Premium ファイル共有は、IO 集中型ワークロードの場合、ほとんどの IO 操作に対して 1 桁台のミリ秒以内で一貫したハイ パフォーマンスと低待機時間を提供しています。 そのため、Premium ファイル共有は、データベース、Web サイトのホスティング、開発環境など、幅広い種類のワークロードに適しています。 Premium ファイル共有は、プロビジョニングされる課金モデルでのみ使用できます。 Premium ファイル共有は、標準的なファイル共有とは切り離されたデプロイ モデルを使用します。

Azure Backup は Premium ファイル共有に対して使用でき、Azure Kubernetes Service はバージョン 1.13 以上で Premium ファイル共有をサポートします。

Premium ファイル共有を作成する方法については、[Azure Premium ファイル ストレージ アカウントの作成方法](storage-how-to-create-premium-fileshare.md)に関する記事を参照してください。

現在、標準ファイル共有と Premium ファイル共有の間で直接変換することはできません。 どちらかの層に切り替える場合は、その層に新しいファイル共有を作成し、作成した新しい共有に元の共有から手動でデータをコピーする必要があります。 これは、Azure Files でサポートされているいずれかのコピー ツール (Robocopy、AzCopy など) を使用して行うことができます。

> [!IMPORTANT]
> Premium ファイル共有は LRS でのみ使用できますが、ストレージ アカウントを提供するほとんどのリージョンで使用できます。 ご自分のリージョンで現在 Premium ファイル共有を使用できるかどうかを見つけるには、Azure の [[リージョン別の利用可能な製品]](https://azure.microsoft.com/global-infrastructure/services/?products=storage) ページを参照してください。

#### <a name="provisioned-shares"></a>プロビジョニングされた共有

Premium ファイル共有は、固定 GiB/IOPS/スループット比に基づいてプロビジョニングされます。 プロビジョニングされた GiB ごとに、共有は、1 IOPS と 0.1 MiB/秒のスループットから、共有ごとの最大限度まで発行されます。 最小許容プロビジョニングは 100 GiB で、最小の IOPS/スループットになります。

ベスト エフォート方式では、すべての共有は、プロビジョニングされたストレージの GiB ごとに 3 IOPS まで、共有のサイズに応じて 60 分またはそれ以上バーストできます。 新しい共有は、プロビジョニングされた容量に基づく完全なバースト クレジットで開始されます。

共有は 1 GiB 単位でプロビジョニングする必要があります。 最小サイズは 100 GiB、次のサイズは 101 GiB、以下同様です。

> [!TIP]
> ベースライン IOPS = 1 * プロビジョニング済み GiB。 (最大 100,000 IOPS まで)。
>
> バースト限度 = 3 * ベースライン IOPS。 (最大 100,000 IOPS まで)。
>
> エグレス レート = 60 MiB/秒 + 0.06 * のプロビジョニング済み GiB
>
> イングレス レート = 40 MiB/秒 + 0.04 * のプロビジョニング済み GiB

プロビジョニングされた共有サイズは、共有クォータによって指定されます。 共有クォータの拡大はいつでも実行できますが、縮小は前回の拡大から 24 時間経過した後にのみ実行できます。 クォータの拡大なしで 24 時間経過した後は、再び拡大するまで、共有クォータを何回でも縮小できます。 IOPS/スループットのスケールの変更は、サイズ変更後、数分以内に有効になります。

プロビジョニングされた共有のサイズを、使用されている GiB 未満に縮小できます。 これを行った場合、データは失われませんが、使用されているサイズに対して引き続き課金され、使用されているサイズではなく、プロビジョニングされた共有のパフォーマンス (ベースライン IOPS、スループット、およびバースト IOPS) を受け取ります。

次の表は、プロビジョニングされた共有サイズについてのこれらの式の例をいくつか示しています。

|容量 (GiB) | ベースライン IOPS | バースト IOPS | エグレス (MiB/秒) | イングレス (MiB/秒) |
|---------|---------|---------|---------|---------|
|100         | 100     | 最大 300     | 66   | 44   |
|500         | 500     | 最大 1,500   | 90   | 60   |
|1,024       | 1,024   | 最大 3,072   | 122   | 81   |
|5,120       | 5,120   | 最大 15,360  | 368   | 245   |
|10,240      | 10,240  | 最大 30,720  | 675 | 450   |
|33,792      | 33,792  | 最大 100,000 | 2,088 | 1,392   |
|51,200      | 51,200  | 最大 100,000 | 3,132 | 2,088   |
|102,400     | 100,000 | 最大 100,000 | 6,204 | 4,136   |

> [!NOTE]
> ファイル共有のパフォーマンスは、他の多くの要因の中でも特にマシン ネットワークの制限、使用可能なネットワーク帯域幅、IO サイズ、並列処理の影響を受けます。 最大のパフォーマンス スケールを達成するには、負荷を複数の VM に分散します。 一般的なパフォーマンスの問題と回避策については、[トラブルシューティング ガイド](storage-troubleshooting-files-performance.md)に関するページを参照してください。

#### <a name="bursting"></a>バースト

Premium ファイル共有は、最大 3 倍の IOPS をバーストできます。 バーストは自動化され、クレジット システムに基づいて動作します。 バーストはベスト エフォートで動作し、バースト限度は保証されるものではなく、ファイル共有は限度*まで*バーストすることができます。

クレジットは、ファイル共有のトラフィックがベースライン IOPS を下回るたびに、バースト バケットに蓄積されます。 たとえば、100 GiB 共有には 100 ベースライン IOPS が含まれます。 共有の実際のトラフィックが特定の 1 秒間で 40 IOPS だった場合は、未使用の 60 IOPS がバースト バケットに補充されます。 これらのクレジットは、後で操作がベースライン IOP を超えたときに使用されます。

> [!TIP]
> バースト バケットのサイズ = ベースライン IOPS * 2 * 3600。

共有は、ベースライン IOPS を超え、バースト バケット内にクレジットがあるときは常にバーストします。 共有はクレジットが残っている限りはバーストを続行できるため、50 TiB より小さい共有のみが最大 1 時間の間、バースト限度で維持されます。 50 TiB より大きい共有は、技術的にはこの 1 時間の限度を超えて最大 2 時間まで可能ですが、これは発生したバースト クレジットの数に基づきます。 ベースライン IOPS を超えた IO のそれぞれが 1 つのクレジットを消費し、すべてのクレジットが消費されると、共有はベースライン IOPS に戻ります。

共有のクレジットには次の 3 つの状態があります。

- 蓄積中 (ファイル共有が使用している量がベースライン IOPS より少ない場合)。
- 減少中 (ファイル共有がバースト中の場合)。
- 定数を維持 (クレジットがないか、ベースライン IOPS が使用中の場合)。

新しいファイル共有は、そのバースト バケットに全数のクレジットが含まれると開始されます。 サーバーによる調整のために共有 IOPS がベースライン IOPS を下回った場合、バースト クレジットは発生しません。

## <a name="file-share-redundancy"></a>ファイル共有の冗長性

Azure Files 標準の共有でサポートされているデータ冗長性オプションは、ローカル冗長ストレージ (LRS)、ゾーン冗長ストレージ (ZRS)、geo 冗長ストレージ (GRS)、geo ゾーン冗長ストレージ (GZRS) (プレビュー) の 4 種類です。

Azure Files の Premium 共有でサポートされるのは、ローカル冗長ストレージ (LRS) だけです。

次のセクションで、さまざまな冗長性オプションの違いについて説明します。

### <a name="locally-redundant-storage"></a>ローカル冗長ストレージ

[!INCLUDE [storage-common-redundancy-LRS](../../../includes/storage-common-redundancy-LRS.md)]

### <a name="zone-redundant-storage"></a>ゾーン冗長ストレージ

[!INCLUDE [storage-common-redundancy-ZRS](../../../includes/storage-common-redundancy-ZRS.md)]

### <a name="geo-redundant-storage"></a>geo 冗長ストレージ

> [!Warning]  
> Azure ファイル共有を GRS ストレージ アカウントのクラウド エンドポイントとして使用している場合は、ストレージ アカウントのフェールオーバーを開始しないでください。 それを行うと、同期の動作が停止し、新しく階層化されたファイルの場合は予期せずデータが失われる可能性があります。 Azure リージョンが失われた場合は、Azure File Sync との互換性のある方法でストレージ アカウントのフェールオーバーがトリガーされます。

geo 冗長ストレージ (GRS) は、プライマリ リージョンから数百マイル離れたセカンダリ リージョンにデータをレプリケートすることで、通年で少なくとも 99.99999999999999% (シックスティーンナイン) の持続性をオブジェクトに確保するように設計されています。 ご使用のストレージ アカウントで GRS が有効になっている場合は、地域的な停電やプライマリ リージョンが復旧できない災害が発生しても、データは保持されます。

読み取りアクセス geo 冗長ストレージ (RA-GRS) を選択する場合は、Azure Files が、現時点ではどのリージョンでも、読み取りアクセス geo 冗長ストレージ (RA-GRS) をサポートしていないことに注意してください。 RA-GRS ストレージ アカウントのファイル共有は GRS アカウントと同じように動作し、GRS 料金で課金されます。

GRS は、セカンダリ リージョンの別のデータセンターにデータをレプリケートしますが、Microsoft がプライマリ リージョンからセカンダリ リージョンへのフェールオーバーを開始する場合にのみ、そのデータを読み取ることができます。

GRS が有効なストレージ アカウントでは、すべてのデータが最初にローカル冗長ストレージ (LRS) でレプリケートされます。 更新は、まずプライマリの場所にコミットされ、LRS を使用してレプリケートされます。 更新は、GRS を使用してセカンダリ リージョンに非同期にレプリケートされます。 データがセカンダリの場所に書き込まれると、LRS を使用してその場所内にレプリケートされます。

プライマリ リージョンとセカンダリ リージョンの両方が、ストレージ スケール ユニット内の異なる障害ドメインとアップグレード ドメイン間でレプリカを管理します。 ストレージ スケール ユニットは、データセンター内の基本的なレプリケーション ユニットです。 このレベルのレプリケーションは LRS で提供されています。詳細については、「[ローカル冗長ストレージ (LRS):Azure Storage の低コストのデータ冗長性](../common/storage-redundancy-lrs.md)」を参照してください。

使用するレプリケーション オプションを決定するときは、次の点に注意してください。

* geo ゾーン冗長ストレージ (GZRS) (プレビュー) では、3 つの Azure 可用性ゾーン間でデータを同期的にレプリケートした後、セカンダリ リージョンに非同期的にデータをレプリケートすることによって、高可用性と最大限の持続性が提供されます。 セカンダリ リージョンへの読み取りアクセスを有効にすることもできます。 GZRS は、オブジェクトに年間 99.99999999999999% (シックスティーンナイン) 以上の持続性を確保するように設計されています。 GZRS の詳細については、[高可用性と最大限の持続性のための geo ゾーン冗長ストレージ (プレビュー)](../common/storage-redundancy-gzrs.md) に関する記事を参照してください。
* ゾーン冗長ストレージ (ZRS) は同期レプリケーションの可用性を高めるため、シナリオによっては GRS よりも適した選択肢となります。 ZRS の詳細については、[ZRS](../common/storage-redundancy-zrs.md) に関するページを参照してください。
* 非同期レプリケーションには、データがプライマリ リージョンに書き込まれた時間から、それがセカンダリ リージョンにレプリケートされるまでの遅延が伴います。 地域的な災害が発生した場合に、データをプライマリ リージョンから復旧できないと、セカンダリ リージョンにまだレプリケートされていない変更が失われる可能性があります。
* GRS では、Microsoft がセカンダリ リージョンへのフェールオーバーを開始しない限り、レプリカを読み取りまたは書き込みアクセスに利用できません。 フェールオーバーの場合、フェールオーバーの完了後にそのデータへの読み取りおよび書き込みアクセス権が得られます。 詳細については、[ディザスター リカバリーのガイダンス](../common/storage-disaster-recovery-guidance.md)に関するページを参照してください。

## <a name="onboard-to-larger-file-shares-standard-tier"></a>大きなファイル共有へのオンボード (Standard レベル)

このセクションは Standard ファイル共有にのみ適用されます。 すべての Premium ファイル共有は、GA オファリングとして 100 TiB を利用できます。

### <a name="restrictions"></a>制限

- Azure プレビューの[使用条件](https://azure.microsoft.com/support/legal/preview-supplemental-terms/)は、Azure ファイル同期デプロイでの使用を含む、プレビュー期間内の大規模なファイル共有に適用されます。
- 新しい汎用ストレージ アカウントを作成する必要があります (既存のストレージ アカウントを拡張することはできません)。
- LRS/ZRS から GRS/GZRS へのアカウント変換は、大きいファイル共有のプレビューへのサブスクリプションが承認された後に作成された新しいストレージ アカウントでは実行できません。


### <a name="regional-availability"></a>リージョン別の提供状況

Standard ファイル共有は、すべてのリージョンで 5 TiB まで利用できます。 一部のリージョンでは、100 TiB の上限まで利用できます。これらのリージョンについては次の表を参照してください。

|リージョン |サポートされる冗長性 |既存のストレージ アカウントをサポートする |ポータルのサポート* |
|-------|---------|---------|---------|
|オーストラリア東部 |LRS     |いいえ    |はい|
|オーストラリア南東部|LRS     |いいえ    |まだ、いいえ|
|インド中部  |LRS     |いいえ    |まだ、いいえ|
|East US        |LRS     |いいえ    |まだ、いいえ|
|フランス中部 |LRS、ZRS|いいえ    |LRS - はい、ZRS - まだ、いいえ|
|フランス南部   |LRS     |いいえ    |はい|
|インド南部    |LRS     |いいえ    |まだ、いいえ|
|東南アジア |LRS、ZRS|いいえ    |はい|
|米国中西部|LRS     |いいえ    |まだ、いいえ|
|西ヨーロッパ    |LRS、ZRS|いいえ    |はい|
|米国西部        |LRS     |いいえ    |まだ、いいえ|
|米国西部 2      |LRS、ZRS|いいえ    |はい|


\* ポータルがサポートされていないリージョンでも、PowerShell または Azure コマンド ライン インターフェイス (CLI) を使用して、5 TiB を超える共有を作成できます。 代わりに、クォータを指定せずに、ポータルを使用して新しい共有を作成します。 これにより、既定のサイズ 100 TiB の共有が作成され、後で PowerShell または Azure CLI を使用して更新できます。

この[アンケート](https://aka.ms/azurefilesatscalesurvey)にご記入ください。新しいリージョンと機能に優先順位を付けるために役立ちます。

### <a name="steps-to-onboard"></a>オンボードの手順

大きなファイル共有プレビューにサブスクリプションを登録するには、Azure PowerShell を使用する必要があります。 [Azure Cloud Shell](https://shell.azure.com/) を使用するか、[Azure PowerShell モジュールをローカルに](https://docs.microsoft.com/powershell/azure/install-Az-ps?view=azps-2.4.0)インストールすることで、次の PowerShell コマンドを実行できます。

まず、プレビューで登録するサブスクリプションが選択されていることを確認します。

```powershell
$context = Get-AzSubscription -SubscriptionId ...
Set-AzContext $context
```

次に、次のコマンドを利用してプレビューに登録します。

```powershell
Register-AzProviderFeature -FeatureName AllowLargeFileShares -ProviderNamespace Microsoft.Storage
Register-AzResourceProvider -ProviderNamespace Microsoft.Storage
```
両方のコマンドを実行すると、サブスクリプションは自動的に承認されます。

登録状態を確認するには、次のコマンドを実行します。

```powershell
Get-AzProviderFeature -FeatureName AllowLargeFileShares -ProviderNamespace Microsoft.Storage
```

状態が "**登録済み**" に更新されるまでに最大 15 分かかる場合があります。 状態が "**登録済み**" になったら、機能を利用できるようになるはずです。

### <a name="use-larger-file-shares"></a>大きなファイル共有を使用する

より大きなファイル共有を使い始めるには、新しい汎用 v2 ストレージ アカウントと新しいファイル共有を作成します。

## <a name="data-growth-pattern"></a>データ増加パターン

現在、Azure ファイル共有の最大サイズは、5 TiB (プレビューでは 100 TiB) です。 この現在の制限のため、Azure ファイル共有をでデプロイするときは、予想されるデータの増加を考慮する必要があります。

Azure ファイル同期を使って複数の Azure ファイル共有を 1 つの Windows ファイル サーバーに同期することができます。これにより、オンプレミスにある古くて大きいファイル共有を Azure File Sync に取り込むことができます。詳しくは、[Azure File Sync のデプロイの計画](storage-files-planning.md)に関するページをご覧ください。

## <a name="data-transfer-method"></a>データ転送方法

オンプレミスのファイル共有などの既存のファイル共有から Azure Files にデータを一括転送するには、多くの簡単なオプションがあります。 以下は一般的な方法の一部です (すべてではありません)。

* **Azure File Sync**:Azure のファイル共有 ("クラウド エンドポイント") と Windows ディレクトリ名前空間 ("サーバー エンドポイント") との間で最初に行われる同期の一部として、Azure File Sync では、すべてのデータが既存のファイル共有から Azure Files にレプリケートされます。
* **[Azure Import/Export](../common/storage-import-export-service.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)** :Azure Import/Export サービスを使うと、ハード ディスク ドライブを Azure データセンターに送付することで、大量のデータを Azure ファイル共有に安全に転送できます。 
* **[Robocopy](https://technet.microsoft.com/library/cc733145.aspx)** :Robocopy は、Windows および Windows Server に付属するよく知られたコピー ツールです。 Robocopy では、ファイル共有をローカルにマウントした後、マウントした場所を Robocopy コマンドのコピー先として使って、Azure Files にデータを転送できます。
* **[AzCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json)** :AzCopy は、最高のパフォーマンスの単純なコマンドを使って Azure Files および Azure Blob Storage との間で双方向にデータをコピーするために設計された、コマンドライン ユーティリティです。

## <a name="next-steps"></a>次の手順
* [Azure File Sync のデプロイの計画](storage-sync-files-planning.md)
* [Azure Files のデプロイ方法](storage-files-deployment-guide.md)
* [Azure ファイル同期のデプロイ方法](storage-sync-files-deployment-guide.md)
