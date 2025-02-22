---
title: Azure File Sync のクラウドの階層化について | Microsoft Docs
description: Azure File Sync のクラウドの階層化について説明します
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 09/21/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 36b09ce8ece010ff24345ddb96654f75542cc9a5
ms.sourcegitcommit: cd70273f0845cd39b435bd5978ca0df4ac4d7b2c
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71098960"
---
# <a name="cloud-tiering-overview"></a>クラウドの階層化の概要
クラウドの階層化は Azure File Sync のオプション機能です。この機能では、頻繁にアクセスされるファイルがサーバー上にローカルにキャッシュされ、その他のファイルはポリシー設定に基づいて Azure Files に階層化されます。 ファイルを階層化すると、Azure File Sync ファイル システム フィルター (StorageSync.sys) がローカルでファイルをポインターと置き換えるか、ポイントを再解析します。 再解析ポイントは Azure Files 内のファイルの URL を表します。 階層化されたファイルをサード パーティ アプリケーションで安全に識別できるように、階層化されたファイルには "オフライン" 属性と FILE_ATTRIBUTE_RECALL_ON_DATA_ACCESS 属性の両方が NTFS 内で設定されます。
 
ユーザーが階層化されたファイルを開くと、Azure File Sync によってファイル データが Azure Files からシームレスに再呼び出しされます。ユーザーは、ファイルが実際に Azure に格納されていることを知る必要はありません。 
 
 > [!Important]  
 > クラウドの階層化は、Windows システム ボリューム上のサーバー エンドポイントではサポートされません。さらに、サイズが 64 KiB より大きいファイルしか Azure Files に階層化することはできません。
    
Azure File Sync では 64 KiB よりも小さいファイルの階層化はサポートされていません。そのようなファイルを階層化および再呼び出することで生じるパフォーマンスのオーバーヘッドが領域の節約を上回るからです。

 > [!Important]  
 > 階層化されたファイルを再呼び出しするには、1 Mbps 以上のネットワーク帯域幅が必要です。 ネットワーク帯域幅が 1 Mbps 未満の場合、タイムアウト エラーでファイルの再呼び出しが失敗する可能性があります。

## <a name="cloud-tiering-faq"></a>クラウドの階層化の FAQ

<a id="afs-cloud-tiering"></a>
### <a name="how-does-cloud-tiering-work"></a>クラウドの階層化のしくみ
Azure File Sync システム フィルターによって、各サーバー エンドポイントでのご利用の名前空間の "ヒートマップ" が作成されます。 さらに、時間の経過と共にアクセス (読み取りおよび書き込み操作) が監視され、アクセスの頻度と新しさの両方に基づいて、すべてのファイルにヒート スコアが割り当てられます。 最近開かれたアクセス頻度の高いファイルはホットと見なされ、一方、ほとんど操作されることがなく、しばらくアクセスされていないファイルはクールと見なされます。 サーバー上のファイル ボリュームが、設定したボリュームの空き領域のしきい値を超えた場合、ご利用の空き領域の割合が満たされるまで最もクールなファイルは Azure Files に階層化されます。

Azure File Sync エージェントのバージョン 4.0 以上では、特定の日数以内にアクセスまたは修正が行われていないファイルを階層化する各サーバー エンドポイントに、日付ポリシーを追加で指定することが可能です。

<a id="afs-volume-free-space"></a>
### <a name="how-does-the-volume-free-space-tiering-policy-work"></a>ボリュームの空き領域の階層化ポリシーのしくみ
ボリュームの空き領域とは、サーバー エンドポイントが配置されているボリューム上に確保する空き領域のサイズです。 たとえば、サーバー エンドポイントが 1 つ含まれているボリューム上でボリュームの空き領域を 20% に設定した場合、最大で 80% のボリューム領域がごく最近アクセスされたファイルによって占有され、この領域に収まりきれない残りのファイルはいずれも Azure に階層化されます。 ボリュームの空き領域は、個々のディレクトリまたは同期グループのレベルではなく、ボリューム レベルで適用されます。 

<a id="volume-free-space-fastdr"></a>
### <a name="how-does-the-volume-free-space-tiering-policy-work-with-regards-to-new-server-endpoints"></a>新しいサーバー エンドポイントに関する、ボリュームの空き領域の階層化ポリシーのしくみ
サーバー エンドポイントが新規にプロビジョニングされ、Azure ファイル共有に接続された場合、サーバーによって最初に名前空間がダウンロードされ、次にそのボリュームの空き領域のしきい値に達するまで、実際のファイルがダウンロードされます。 このプロセスは、高速ディザスター リカバリーまたは名前空間の迅速な復元とも呼ばれます。

<a id="afs-effective-vfs"></a>
### <a name="how-is-volume-free-space-interpreted-when-i-have-multiple-server-endpoints-on-a-volume"></a>ボリューム上に複数のサーバー エンドポイントがある場合、ボリュームの空き領域はどのように解釈されますか。
ボリューム上に複数のサーバー エンドポイントがある場合、ボリュームの空き領域の有効なしきい値は、そのボリューム上のサーバー エンドポイントで指定されているボリュームの最大空き領域になります。 ファイルは、どのサーバー エンドポイントに属しているかに関係なく、使用パターンに従って階層化されます。 たとえば、ボリューム上に Endpoint1 と Endpoint2 の 2 つのサーバー エンドポイントがあり、ボリュームの空き領域のしきい値が Endpoint1 では 25%、Endpoint2 では 50% の場合、ボリュームの空き領域のしきい値は、どちらのサーバー エンドポイントでも 50% になります。 

<a id="date-tiering-policy"></a>
### <a name="how-does-the-date-tiering-policy-work-in-conjunction-with-the-volume-free-space-tiering-policy"></a>日付の階層化ポリシーを、ボリューム空き領域の階層化ポリシーと組み合わせて動作させるには、どうしたらいいですか。 
サーバー エンドポイント上でクラウドの階層化を有効にする場合は、ボリューム空き領域ポリシーを設定します。 このポリシーは、日付ポリシーなどの他のポリシーよりも常に優先されます。 必要に応じて、該当のボリューム上の各サーバー エンドポイントの日付ポリシーを有効にできます。有効にすると、このポリシーが示す日付範囲内にアクセス (つまり、読み取りまたは書き込み) があったファイルだけが、古いファイルを階層化した状態で、ローカルに保持されます。 ボリュームの空き領域ポリシーが常に優先されること、また、ファイルの価値に見合う日付ポリシーに示された日数を保持しておく十分な空き領域がない場合、Azure File Sync はボリューム空き領域の割合が適切になるまで、最もアクセスの少ないファイルの階層化を続けることを念頭に置いてください。

たとえば、日付ベースの階層化ポリシーが 60 日間で、ボリューム空き領域ポリシーが 20% であるとします。 日付ポリシーを適用した後に、ボリューム上の空き領域が 20% 未満になった場合、ボリューム空き領域ポリシーが適用されて日付ポリシーをオーバーライドします。 これにより、サーバー上に保持されるデータ量が 60 日分から 45 日分に減少し、より多くのファイルが階層化されることになります。 逆に言えば、このポリシーでは、空き領域のしきい値に到達していない場合でも、時間範囲外にあるファイルの階層化を実行します。そのため、ボリュームが空であっても、61 日が経過しているファイルは階層化されます。

<a id="volume-free-space-guidelines"></a>
### <a name="how-do-i-determine-the-appropriate-amount-of-volume-free-space"></a>ボリュームの空き領域の適切なサイズを決定するにはどうすればよいですか。
ローカルに保持しておく必要があるデータの量は、ご利用の帯域幅、データセットのアクセス パターン、および予算などいくつかの要因によって決定されます。 低帯域幅の接続を利用している場合は、ユーザーに対する遅延を確実に最小限に抑えるためにより多くのデータをローカルに保持することをお勧めします。 それ以外の場合は、特定の期間中のチャーン レートを基に決めることができます。 たとえば、毎月、ご利用の 1 TB データセットの約 10% が変更されているか、または活発にアクセスされることを把握している場合は、ファイルの再呼び出しを頻繁に行わなくても済むように 100 GB をローカルに保持することをお勧めします。 ご利用のボリュームが 2 TB である場合は、5% (または 100 GB) をローカルに保持します。つまり、残りの 95% がボリュームの空き領域の割合となります。 ただし、チャーン レートがより高い期間を考慮してバッファーを追加することをお勧めします。つまり、より低い、ボリュームの空き領域の割合から始めて、後で必要に応じてその割合を調整するということです。 

ローカルに保持するデータを増やせば、Azure から再呼び出しするファイル数が少なくなるので、エグレス コストは減少します。しかし、維持しなければならないオンプレミスのストレージの量が大きくなり、独自のコストがかかることになります。 Azure File Sync のインスタンスがデプロイされたら、ご利用のストレージ アカウントのエグレスを確認することで、ボリュームの空き領域の設定が自分の使用状況に適しているかどうかをおおまかに判断することができます。 ストレージ アカウントにご利用の Azure File Sync クラウド エンドポイント (つまり、ご利用の同期共有) のみが含まれていると仮定した場合、高いエグレスは、多くのファイルがクラウドから再呼び出しされていることを意味するので、ローカル キャッシュを増やすことを検討する必要があります。

<a id="how-long-until-my-files-tier"></a>
### <a name="ive-added-a-new-server-endpoint-how-long-until-my-files-on-this-server-tier"></a>新しいサーバー エンドポイントを追加しました。 このサーバー上のファイルが階層化されるまで、どのくらいの期間かかりますか。
Azure File Sync エージェントのバージョン 4.0 以上では、ファイルが Azure ファイル共有にアップロードされると、以降は、次の階層化セッションが実行されるとすぐに、ポリシーに従って階層化されます。これは、1 時間ごとに行われます。 古いエージェントでは、階層化が実行されるまでに、最大で 24 時間かかる場合があります。

<a id="is-my-file-tiered"></a>
### <a name="how-can-i-tell-whether-a-file-has-been-tiered"></a>ファイルが階層化されているかどうかは、どうやって判断できますか。
ファイルが Azure ファイル共有に階層化されたかどうかを確認するには、いくつかの方法があります。
    
   *  **ファイル上でファイル属性を確認します。**
     ファイルを右クリックして **[詳細]** に移動し、 **[属性]** プロパティまで下へスクロールします。 階層化されたファイルには、次のような属性設定があります。     
        
        | 属性の文字 | 属性 | 定義 |
        |:----------------:|-----------|------------|
        | A | アーカイブ | バックアップ ソフトウェアによって、ファイルがバックアップされる必要があることを示します。 この属性は、ファイルが階層化されているか、ディスク上に完全に格納されているかに関係なく、常に設定されます。 |
        | P | スパース ファイル | ファイルがスパース ファイルであることを示します。 スパース ファイルとは、ディスク ストリーム上のファイルがほぼ空である場合に、効率的に使用するために NTFS が提供している特別な種類のファイルです。 ファイルは完全に階層化されているか、部分的に再現されているため、Azure File Sync ではスパース ファイルが使用されます。 完全に階層化されたファイルでは、ファイル ストリームがクラウドに格納されます。 部分的に再現されているファイルでは、ファイルの一部が既にディスク上に存在します。 ファイルがディスク上に完全に再現されている場合、Azure File Sync では、ファイルはスパース ファイルから通常のファイルに変換されます。 |
        | L | 再解析ポイント | ファイルが再解析ポイントを保持していることを示します。 再解析ポイントは、ファイル システム フィルターによって使用される特殊なポインターです。 Azure File Sync では、ファイルが格納されるクラウドの場所を Azure File Sync のファイル システム フィルター (StorageSync.sys) に対して定義するために再解析ポイントを使用します。 これにより、シームレスなアクセスが可能になります。 Azure File Sync が使用されていることや、Azure ファイル共有に格納されているファイルへのアクセス方法をユーザーが知る必要はありません。 ファイルが完全に再現されている場合、Azure File Sync によって、そのファイルから再解析ポイントが削除されます。 |
        | O | オフライン | ファイルのコンテンツの一部またはすべてがディスク上に保存されていないことを示します。 ファイルが完全に再現されている場合、Azure File Sync によってこの属性は削除されます。 |

        ![[詳細] タブが選択された、ファイルの [プロパティ] ダイアログ ボックス](media/storage-files-faq/azure-file-sync-file-attributes.png)
        
        また、**属性**フィールドをエクスプローラーのテーブル表示に追加することで、フォルダー内にあるすべてのファイルの属性を確認できます。 これを行うには、既存の列 ( **[サイズ]** など) を右クリックして、 **[詳細]** を選択し、ドロップダウン リストから **[属性]** を選択します。
        
   * **`fsutil` を使用して、ファイル上の再解析ポイントを確認します。**
       前記のオプションで説明したように、階層化されたファイルには必ず再解析ポイントが設定されます。 再解析ポインターは、Azure File Sync のファイル システム フィルター (StorageSync.sys) の特別なポインターです。 ファイルに再解析ポイントがあるかどうかを調べるには、管理者特権でのコマンド プロンプトまたは PowerShell ウィンドウで、`fsutil` ユーティリティを実行します。
    
        ```powershell
        fsutil reparsepoint query <your-file-name>
        ```

        ファイルに再解析ポイントがある場合、"**Reparse Tag Value: 0x8000001e**" と表示されることが想定されます。 この 16 進値は、Azure File Sync が所有する再解析ポイントの値です。また、出力には、Azure ファイル共有上のファイルへのパスを表す再解析データが含まれます。

        > [!WARNING]  
        > `fsutil reparsepoint` ユーティリティ コマンドには、再解析ポイントを削除する機能も含まれています。 Azure File Sync のエンジニア チームによって指示されない限り、このコマンドは実行しないでください。 このコマンドを実行すると、データが失われる可能性があります。 

<a id="afs-recall-file"></a>

### <a name="a-file-i-want-to-use-has-been-tiered-how-can-i-recall-the-file-to-disk-to-use-it-locally"></a>使用したいファイルが階層化されています。 ローカルで使用するためにこのファイルをディスクに再現するには、どうすればよいですか。
ディスクにファイルを再現する最も簡単な方法は、ファイルを開くことです。 Azure File Sync のファイル システム フィルター (StorageSync.sys) は、Azure ファイル共有からファイルをシームレスにダウンロードします。ユーザー側で作業を行う必要はありません。 マルチメディア ファイルや .zip ファイルなど、部分的に読み取りできるファイルの種類の場合、ファイルを開いても、ファイル全体がダウンロードされることはありません。

また、PowerShell を使ってファイルを強制的に再現することも可能です。 複数のファイル (フォルダー内にあるすべてのファイルなど) を一度に再現したい場合に、この方法が役立つことがあります。 Azure File Sync がインストールされているサーバー ノードへの PowerShell セッションを開き、次の PowerShell コマンドを実行します。
    
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncFileRecall -Path <file-or-directory-to-be-recalled>
```

<a id="sizeondisk-versus-size"></a>
### <a name="why-doesnt-the-size-on-disk-property-for-a-file-match-the-size-property-after-using-azure-file-sync"></a>ファイルの "*ディスク上のサイズ*" プロパティが、Azure File Sync を使用した後の "*サイズ*" プロパティと一致しないのはどうしてですか。 
Windows のエクスプローラーでは、 **[サイズ]** と **[ディスク上のサイズ]** という、ファイルのサイズを表現する 2 つのプロパティを公開しています。 これらのプロパティは、意味が微妙に異なります。 **[サイズ]** は、ファイルそのもののサイズを表します。 **[ディスク上のサイズ]** は、ディスクに格納されているファイル ストリームのサイズを表します。 圧縮、データ重複除去の使用、Azure File Sync の使用によるクラウド階層化など、さまざまな理由から、これらのプロパティの値が異なる場合があります。ファイルが Azure ファイル共有に階層化された場合、ディスク上ではなく Azure ファイル共有にファイル ストリームが格納されるため、ディスク上のサイズはゼロになります。 また、ファイルが部分的に階層化される (または、部分的に再現される) 場合もあります。 部分的に階層化されたファイルでは、ファイルの一部がディスク上に存在します。 これは、マルチメディア プレーヤーや圧縮ユーティリティなどのアプリケーションによってファイルが部分的に読み取られた場合に、発生する可能性があります。 

<a id="afs-force-tiering"></a>
### <a name="how-do-i-force-a-file-or-directory-to-be-tiered"></a>ファイルまたはディレクトリを強制的に階層化するには、どうすればよいですか。
クラウドの階層化機能が有効な場合は、クラウド エンドポイントに指定されたボリューム空き領域の割合を達成するために、最終アクセス時刻と最終変更時刻に基づいてファイルが自動的に階層化されます。 ただし、手動で強制的にファイルを階層化する必要がある場合もあります。 長期間再使用する予定がない大きなファイルを保存して、当面ボリューム上に他のファイルとフォルダーのための領域を空けておきたい場合、手動による階層化が役立つ可能性があります。 次の PowerShell コマンドを使って、強制的に階層化できます。

```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Invoke-StorageSyncCloudTiering -Path <file-or-directory-to-be-tiered>
```

## <a name="next-steps"></a>次の手順
* [Azure File Sync のデプロイの計画](storage-sync-files-planning.md)
