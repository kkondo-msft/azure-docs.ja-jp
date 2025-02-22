---
title: Azure HPC キャッシュを作成する
description: Azure HPC Cache インスタンスを作成する方法
author: ekpgh
ms.service: hpc-cache
ms.topic: tutorial
ms.date: 09/06/2019
ms.author: v-erkell
ms.openlocfilehash: e1b69f17d964647944f23f4d16a0a1a5f112b60d
ms.sourcegitcommit: 0fab4c4f2940e4c7b2ac5a93fcc52d2d5f7ff367
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/17/2019
ms.locfileid: "71037068"
---
# <a name="create-an-azure-hpc-cache"></a>Azure HPC キャッシュを作成する

Azure portal を使用してキャッシュを作成します。

![Azure portal に表示されるキャッシュの概要と下部の作成ボタンのスクリーンショット](media/hpc-cache-home-page.png)

## <a name="define-basic-details"></a>基本的な詳細を定義する

![Azure portal のプロジェクト詳細ページのスクリーンショット](media/hpc-cache-create-basics.png)

**[プロジェクトの詳細]** で、Azure HPC キャッシュのホストとなるサブスクリプションとリソース グループを選択します。 [プレビュー アクセス](hpc-cache-prereqs.md#azure-subscription) リストにサブスクリプションが登録されていることを確認してください。

**[サービスの詳細]** で、キャッシュの名前と、これらのその他の属性を設定します。

* 場所 - [サポートされているリージョン](hpc-cache-overview.md#region-availability)のいずれかを選択します。
* 仮想ネットワーク - 既存のものを選択するか、新しい仮想ネットワークを作成することができます。
* サブネット - Azure HPC キャッシュ専用に少なくとも 64 個の IP アドレス (/24) を含むサブネットを選択または作成します。

## <a name="set-cache-capacity"></a>キャッシュ容量を設定する
<!-- change link in GUI -->

**[キャッシュ]** ページで、Azure HPC キャッシュの容量を設定する必要があります。 ご自分のキャッシュで保持できるデータの量とクライアント要求の処理の迅速さは、この値によって決まります。 パブリック プレビュー期間後は、容量もキャッシュのコストに影響します。

キャッシュ容量は、1 秒あたりの入出力操作 (IOPS) で測定されます。 これらの 2 つの値を設定して容量を選択してください。

* キャッシュの最大データ転送速度 (スループット) (GB/秒)
* キャッシュ データ用に割り当てるストレージの容量 (TB)

選択可能ないずれかのスループット値とキャッシュ ストレージ サイズを選択してください。 IOPS 容量が計算されて値セレクターの下に表示されます。

実際のデータ転送速度は、ワークロードとネットワーク速度、ストレージ ターゲットの種類によって異なることに注意してください。 ファイルがキャッシュに存在しない (または古いものとしてマークされている) 場合は、それをバックエンド ストレージからフェッチするために一部のスループットが使用されます。 選択した値によって設定されるのはキャッシュ全体の最大スループットであり、そのすべてをクライアント要求に使用できるわけではありません。

キャッシュ ヒット率を最大限に高めるために、Azure HPC Cache ではキャッシュ ストレージに関して、どのファイルをキャッシュして事前に読み込むかを管理します。 キャッシュの内容は絶えず評価され、アクセスされる頻度が低くなったファイルは長期ストレージに移されます。 メタデータ用の追加領域やその他のオーバーヘッドと共にアクティブな作業ファイル一式を無理なく保持できるキャッシュ ストレージ サイズを選んでください。

![キャッシュ サイズ設定ページのスクリーンショット](media/hpc-cache-create-iops.png)

## <a name="add-storage-targets"></a>ストレージ ターゲットを追加する

ストレージ ターゲットは、ご自分のキャッシュの内容を格納する長期バックエンド ストレージです。

ストレージ ターゲットはキャッシュの作成中に定義することもできますが、ポータルにあるご自分のキャッシュのページの **[構成]** セクションでリンクを使用して、それらを後から追加することもできます。

![ストレージ ターゲットのページのスクリーンショット](media/hpc-cache-storage-targets-pop.png)

**[ストレージ ターゲットの追加] リンク**をクリックして、バックエンド ストレージ システムを定義します。 ストレージには、Azure BLOB コンテナーまたはオンプレミスの NFS システムを使用できます。

最大 10 個の異なるストレージ ターゲットを定義できます。

ストレージ ターゲットを追加する具体的な手順については、[ストレージの追加](hpc-cache-add-storage.md)に関するページを参照してください。 その手順は、BLOB ストレージと NFS エクスポートで異なります。

いくつかのヒントを次に示します。 

* どちらのタイプのストレージも、バックエンド ストレージ システムの検出方法 (NFS アドレスまたは BLOB コンテナー名) とクライアント側の名前空間のパスを指定する必要があります。

* BLOB ストレージ ターゲットを作成する際は、[アクセス制御ロールの追加](hpc-cache-add-storage.md#add-the-access-control-roles-to-your-account)に関するセクションの説明のとおり、キャッシュにストレージ アカウントへのアクセス許可があることを確認してください。 ロールの構成を正しくできるかどうかが不安な場合は、最初にキャッシュを作成した後で BLOB ストレージを追加してください。

* NFS ストレージ ターゲットを作成するときは、[使用モデル](hpc-cache-add-storage.md#choose-a-usage-model)を指定します。 使用モデルの設定が、キャッシュによるワークフローの最適化に役立ちます。

## <a name="add-resource-tags-optional"></a>リソース タグを追加する (省略可)

**[タグ]** ページでは、ご自分の Azure HPC キャッシュに[リソース タグ](https://go.microsoft.com/fwlink/?linkid=873112)を追加できます。 

## <a name="finish-creating-the-cache"></a>キャッシュの作成を完了する

新しいキャッシュを構成したら、 **[確認と作成]** タブをクリックします。選択内容がポータルによって検証されるほか、自分で選択内容を確認することができます。 すべて正しければ **[作成]** をクリックしてください。 

キャッシュの作成には 10 分程度かかります。 進行状況は、Azure portal の通知パネルで追跡できます。 

![ポータルのキャッシュ作成画面 ("デプロイが進行中です" ページと "通知" ページ) のスクリーンショット](media/hpc-cache-deploy-status.png)

作成が完了すると、新しい Azure HPC Cache インスタンスへのリンクと共に通知が表示され、ご利用のサブスクリプションの **[リソース]** リストにキャッシュが表示されます。 

![Azure portal における Azure HPC Cache インスタンスのスクリーンショット](media/hpc-cache-new-overview.png)

## <a name="next-steps"></a>次の手順

**[リソース]** 一覧に自分のキャッシュが表示されたら、クライアントがアクセスできるようにそれをマウントしたり、それを使用してワーキング セットのデータを新しい Azure Blob Storage ターゲットに移動したり、別のデータ ソースを定義したりすることができます。

* [Azure HPC キャッシュをマウントする](hpc-cache-mount.md)
* [Azure HPC Cache の Azure Blob Storage にデータを移動する](hpc-cache-ingest.md)
* [ストレージ ターゲットを追加する](hpc-cache-add-storage.md)
