---
title: Azure HPC Cache クラウド コンテナーにデータを移動する
description: Azure HPC Cache で使用する Azure Blob Storage にデータを事前設定する方法
author: ekpgh
ms.service: hpc-cache
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: v-erkell
ms.openlocfilehash: 07a97b1afa8049ace97f1589393cd76c24f21368
ms.sourcegitcommit: a4b5d31b113f520fcd43624dd57be677d10fc1c0
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/06/2019
ms.locfileid: "70775167"
---
# <a name="move-data-to-azure-blob-storage-for-azure-hpc-cache"></a>Azure HPC Cache の Azure Blob Storage にデータを移動する

Azure Blob Storage へのデータの移動がワークフローに含まれる場合は、Azure HPC Cache 経由でデータをコピーする効率的な方法を使用していることを確認してください。

この記事では、Azure HPC Cache で使用する Blob Storage にデータを移動する最良の方法について説明します。

次の情報に留意してください。

* Azure HPC Cache では、Blob Storage 内のデータを整理するために、専用のストレージ形式が使用されます。 Blob Storage ターゲットが、新しいコンテナーか空のコンテナー、またはそれまで Azure HPC Cache データに使用されていた BLOB コンテナーのいずれかでなければならないのは、そのためです。 このクラウド ファイル システムは、[Avere vFXT for Azure](https://azure.microsoft.com/services/storage/avere-vfxt/) でも使用されます。

* Azure HPC Cache 経由でデータをコピーするのが最善の選択肢となるのは、複数のクライアントおよび並列操作を使用するときです。 1 つのクライアントからの単純なコピー コマンドでは、データの移動が低速になります。

Blob Storage コンテナーにコンテンツを読み込む Python ベースのユーティリティが利用できます。 詳細については、[Blob Storage へのデータの事前読み込み](#pre-load-data-in-blob-storage-with-clfsload)のセクションを参照してください。

読み込みユーティリティを使用したくない場合、または既存のストレージ ターゲットにコンテンツを追加したい場合は、「[Azure HPC Cache 経由でデータをコピーする](#copy-data-through-the-azure-hpc-cache)」にある並列データ取り込み処理のヒントを参考にしてください。 

## <a name="pre-load-data-in-blob-storage-with-clfsload"></a>CLFSLoad を使用して Blob Storage にデータを事前に読み込む

新しい Blob Storage コンテナーをストレージ ターゲットとして追加する前に、[Avere CLFSLoad](https://aka.ms/avere-clfsload) ユーティリティを使用して、Blob Storage コンテナーにデータをコピーすることができます。 このユーティリティは Linux VM 上で動作し、Azure HPC Cache に必要な独自の形式でデータを書き込みます。 キャッシュで使用する Blob Storage コンテナーにデータを事前設定する方法としては、これが最も効率的です。

この方法が使用できるのは新しい空のコンテナーのみです。 コンテナーは、Avere CLFSLoad を使用する前に作成してください。

詳細については、[Avere CLFSLoad の Readme](https://github.com/microsoft/Avere-CLFSLoad/blob/master/README.md) を参照してください。 <!-- caution literal link -->

大まかな手順を以下に示します。

1. Python バージョン 3.6 以降がインストールされた Linux システム (物理マシンまたは VM) を用意します (パフォーマンスを高めるために Python 3.7 を推奨します)。
1. Avere-CLFSLoad ソフトウェアを Linux システムにインストールします。
1. Linux のコマンド ラインから転送を実行します。

Avere CLFSLoad ユーティリティには、次の情報が必要です。

* Blob Storage コンテナーが存在するストレージ アカウントの ID
* 空の Blob Storage コンテナーの名前
* コンテナーへの書き込みをユーティリティに許可する Shared Access Signature (SAS) トークン
* データ ソースへのローカル パス (コピーするデータが格納されたローカル ディレクトリ、またはデータが格納されているマウントされたリモート システムのローカル パス)

これらの要件については、[Avere CLFSLoad の Readme](https://aka.ms/avere-clfsload) に詳しく記載されています。

## <a name="copy-data-through-the-azure-hpc-cache"></a>Azure HPC Cache 経由でデータをコピーする

Avere CLFSLoad ユーティリティを使用したくない場合、または大量のデータを既存の Blob Storage ターゲットに追加したい場合は、キャッシュ経由でデータをコピーすることができます。 Azure HPC Cache は、複数のクライアントからの要求を同時に処理するように設計されています。したがって、キャッシュ経由でデータをコピーするには、複数クライアントからの並列書き込みを使用する必要があります。

![マルチクライアントのマルチスレッドのデータ移動を示す図:左上に、オンプレミスのハードウェア ストレージを表すアイコンがあり、そこから複数の矢印が出ています。 矢印は、4 つのクライアント マシンを指しています。 各クライアント マシンから Azure HPC Cache に向かって 3 本の矢印が出ています。 Azure HPC Cache から、複数の矢印が Blob Storage を指しています。](media/hpc-cache-parallel-ingest.png) 

ストレージ システム間でデータを転送するためによく使われる ``cp`` または ``copy`` コマンドは、一度に 1 つのファイルだけをコピーするシングルスレッドのプロセスです。 つまり、一度に 1 つのファイルしかファイル サーバーに取り込まれません。これでは、クラスターのリソースを浪費してしまいます。

このセクションでは、Azure HPC Cache の Blob Storage にデータを移動するための、マルチクライアント、マルチスレッドのファイル コピー システムを作成する方法について説明します。 複数のクライアントとシンプルなコピー コマンドを使ってデータを効率よくコピーするために使用できる、ファイル転送の概念と決定点について説明します。

便利なユーティリティについても説明します。 ``msrsync`` ユーティリティを使用すると、データセットを複数のバケットに分割したうえで rsync コマンドを使用するというプロセスを部分的に自動化できます。 もう 1 つ、``parallelcp`` スクリプトというユーティリティがあります。これはソース ディレクトリを読み取り、自動的にコピー コマンドを発行するユーティリティです。

### <a name="strategic-planning"></a>戦略的計画

データを並列コピーするための戦略を作成するときは、ファイルのサイズ、ファイルの数、ディレクトリの深さにおけるトレードオフを理解する必要があります。

* ファイルが小さいとき、注目すべきメトリックは 1 秒あたりのファイル数です。
* ファイルが大きい場合 (10 MiBi 以上)、注目すべきメトリックは 1 秒あたりのバイト数です。

コピー プロセスごとにスループット率とファイル転送された率が示されます。これらは、コピー コマンドの長さのタイミングを計り、ファイル サイズとファイル数を考慮することによって、測定できます。 それらの測定方法についての説明は、このドキュメントの範囲外ですが、小さいファイルを処理するのか、大きいファイルを処理するのかを把握することが不可欠です。

Azure HPC Cache を使用した並列データ取り込みには、次の方法があります。

* 手動コピー - クライアント上で、マルチスレッドのコピーを手動で作成できます。そのためには、定義済みのファイル セットまたはパスのセットに対して、一度に複数のコピー コマンドをバックグラウンドで実行します。 詳細については、[Azure HPC Cloud のデータ取り込み (手動でコピーする方法)](hpc-cache-ingest-manual.md) に関するページを参照してください。

* ``msrsync`` を使用して部分的に自動化されたコピー  -  ``msrsync`` は、複数の並列 ``rsync`` プロセスを実行するラッパー ユーティリティです。 詳細については、「[Azure HPC Cache のデータ取り込み - msrsync を使用した方法](hpc-cache-ingest-msrsync.md)」を参照してください。

* コピー スクリプト (``parallelcp``) の使用 - 並列コピー スクリプトの作成と実行の方法については、「[Azure HPC Cache のデータ取り込み - 並列コピー スクリプトを使用した方法](hpc-cache-ingest-parallelcp.md)」を参照してください。

## <a name="next-steps"></a>次の手順

ストレージのセットアップ後にクライアントからキャッシュをマウントする方法を確認しましょう。

* [Azure HPC Cache システムにアクセスする](hpc-cache-mount.md)
