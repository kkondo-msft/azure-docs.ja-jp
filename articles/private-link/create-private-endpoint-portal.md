---
title: Azure でプライベート エンドポイントを管理する
description: Azure portal を使用してプライベート エンドポイントを作成する方法を説明します。
services: private-link
author: KumudD
ms.service: private-link
ms.topic: quickstart
ms.date: 09/16/2019
ms.author: kumud
ms.openlocfilehash: 2b7ad95dc652f2099af75308f05ddd0684e088aa
ms.sourcegitcommit: 1c9858eef5557a864a769c0a386d3c36ffc93ce4
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/18/2019
ms.locfileid: "71104743"
---
# <a name="create-a-private-endpoint-using-azure-portal"></a>Azure portal を使用してプライベート エンドポイントを作成する

プライベート エンドポイントは、Azure におけるプライベート リンクの基本的な構成要素です。 これによって、仮想マシン (VM) などの Azure リソースと Private Link リソースとの非公開での通信が可能になります。 このクイックスタートでは、Azure PowerShell を使用して Azure Virtual Network 上に VM、SQL Database サーバー、Azure プライベート エンドポイントを作成する方法について説明します。 その後、VM から SQL Database サーバーに安全にアクセスできます。

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

## <a name="sign-in-to-azure"></a>Azure へのサインイン

Azure Portal ( https://portal.azure.com ) にサインインします。

## <a name="create-a-vm"></a>VM の作成
このセクションでは、Private Link リソース (この例では、Azure 上の SQL サーバー) へのアクセスに使用する VM をホストするために、仮想ネットワークとサブネットを作成します。

### <a name="create-the-virtual-network"></a>仮想ネットワークの作成


このセクションでは、Private Link リソースへのアクセスに使用する VM をホストするために、仮想ネットワークとサブネットを作成します。

1. 画面の左上で、 **[リソースの作成]**  >  **[ネットワーキング]**  >  **[仮想ネットワーク]** の順に選択します。
1. **[仮想ネットワークの作成]** に次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    | 名前 | 「*MyVirtualNetwork*」と入力します。 |
    | アドレス空間 | 「*10.1.0.0/16*」を入力します。 |
    | Subscription | サブスクリプションを選択します。|
    | Resource group | **[新規作成]** を選択し、「*myResourceGroup*」と入力して、 **[OK]** を選択します。 |
    | Location | **[WestCentralUS]** を選択します。|
    | サブネット - 名前 | 「*mySubnet*」と入力します。 |
    | サブネット アドレス範囲 | 「*10.1.0.0/24*」と入力します。 |
    |||
1. 残りは既定値のままにして、 **[作成]** を選択します。


### <a name="create-virtual-machine"></a>仮想マシンの作成

1. Azure portal の画面の左上で、 **[リソースの作成]**  >  **[Compute]**  >  **[仮想マシン]** を選択します。

1. **[仮想マシンの作成 - 基本]** に次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    | **プロジェクトの詳細** | |
    | Subscription | サブスクリプションを選択します。 |
    | Resource group | **[myResourceGroup]** を選択します。 これは前のセクションで作成しました。  |
    | **インスタンスの詳細** |  |
    | 仮想マシン名 | 「*myVm*」と入力します。 |
    | リージョン | **[WestCentralUS]** を選択します。 |
    | 可用性のオプション | 既定値 **[インフラストラクチャ冗長は必要ありません]** をそのまま使用します。 |
    | Image | **[Windows Server 2019 Datacenter]** を選択します。 |
    | Size | 既定値 **[Standard DS1 v2]** をそのまま使用します。 |
    | **管理者アカウント** |  |
    | ユーザー名 | 任意のユーザー名を入力します。 |
    | パスワード | 任意のパスワードを入力します。 パスワードは 12 文字以上で、[定義された複雑さの要件](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm)を満たす必要があります。|
    | パスワードの確認 | パスワードを再入力します。 |
    | **受信ポートの規則** |  |
    | パブリック受信ポート | 既定値 **[なし]** のままにします。 |
    | **コスト削減** |  |
    | Windows ライセンスを既にお持ちの場合 | 既定値 **[なし]** のままにします。 |
    |||

1. **[次へ:ディスク]** を選択します。

1. **[仮想マシンの作成 - Disk]** で、既定値のままにし、 **[Next: Networking]\(次へ : ネットワーク\)** を選択します。

1. **[Create a virtual machine - Networking]\(仮想マシンの作成 - ネットワーク\)** で次の情報を選択します。

    | Setting | 値 |
    | ------- | ----- |
    | 仮想ネットワーク | 既定値 **[MyVirtualNetwork]** のままにします。  |
    | アドレス空間 | 規定値 **[10.1.0.0/24]** のままにします。|
    | Subnet | 規定値 **[mySubnet (10.1.0.0/24)]** のままにします。|
    | パブリック IP | 既定値 **(new) myVm-ip\((新規) myVm-ip** のままにします。 |
    | パブリック受信ポート | **[選択したポートを許可する]** を選択します。 |
    | 受信ポートの選択 | **[HTTP]** と **[RDP]** を選択します。|
    |||


1. **[Review + create]\(レビュー + 作成\)** を選択します。 **[確認および作成]** ページが表示され、Azure によって構成が検証されます。

1. "**証に成功しました**" というメッセージが表示されたら、 **[作成]** を選択します。

## <a name="create-a-sql-database-server"></a>SQL データベース サーバーの作成
このセクションでは、Azure で SQL データベース サーバーを作成します。 

1. Azure portal の画面の左上で、 **[リソースの作成]**  >  **[データベース]**  >  **[SQL データベース]** を選択します。

1. **[SQL データベースの作成 - 基本]** で次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    | **データベースの詳細** | |
    | Subscription | サブスクリプションを選択します。 |
    | Resource group | **[myResourceGroup]** を選択します。 これは前のセクションで作成しました。|
    | **インスタンスの詳細** |  |
    | データベース名  | 「*mydatabase*」と入力します。 この名前を取得する場合は、一意の名前を作成します。 |
    |||
5.  **[サーバー]** で、 **[新規作成]** を選択します。 
6.  **[新しいサーバー]** で次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    |サーバー名  | 「*myserver*」と入力します。 この名前を取得する場合は、一意の名前を作成します。|
    | サーバー管理者のログイン| 任意の管理者名を入力します。 |
    | パスワード | 任意のパスワードを入力します。 パスワードは 8 文字以上で、定義された要件を満たす必要があります。 |
    | Location | SQL サーバーを配置する Azure リージョンを選択します。 |
    
7.  **[OK]** を選択します。 
8.  **[確認および作成]** を選択します。  **[確認および作成]**   ページが表示され、Azure によって構成が検証されます。 
9. "検証に成功しました" というメッセージが表示されたら、 **[作成]** を選択します。 
10. "検証に成功しました" というメッセージが表示されたら、[作成] を選択します。 

## <a name="create-a-private-endpoint"></a>プライベート エンドポイントの作成

このセクションでは、SQL サーバーを作成し、それにプライベート エンドポイントを追加します。 

1. Azure portal の画面の左上で、 **[リソースの作成]**  >  **[ネットワーキング]**  >  **[プライベート リンク センター (プレビュー)]** を選択します。
2.  **[プライベート リンク センター - 概要]** の **サービスへのプライベート接続を構築する**オプションで、 **[開始]** を選択します。
1. **[Create a private endpoint (Preview) - Basics]\(プライベート エンドポイント (プレビュー) の作成 - 基本\)** で次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    | **プロジェクトの詳細** | |
    | Subscription | サブスクリプションを選択します。 |
    | Resource group | **[myResourceGroup]** を選択します。 これは前のセクションで作成しました。|
    | **インスタンスの詳細** |  |
    | 名前 | 「 * myPrivateEndpoint*」と入力します。 この名前を取得する場合は、一意の名前を作成します。 |
    |リージョン|**[WestCentralUS]** を選択します。|
    |||
5.  **[次へ: リソース]** を選択します。
6.  **[Create a private endpoint - Resource]\(プライベート エンドポイントの作成 - リソース\)** で次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    |接続方法  | 自分のディレクトリ内の Azure リソースに接続するように選択します。|
    | Subscription| サブスクリプションを選択します。 |
    | リソースの種類 |  **[Microsoft.Sql/servers]** を選択します。 |
    | リソース |*[myServer]* を選択します。|
    |ターゲット サブリソース |*[sqlServer]* を選択します。|
    |||
7.  **[次へ: 構成]** を選択します。
8. **[Create a private endpoint (Preview) - Configuration]\(プライベート エンドポイント (プレビュー) の作成 - 構成\)** で次の情報を入力または選択します。

    | Setting | 値 |
    | ------- | ----- |
    |**ネットワーキング**| |
    | 仮想ネットワーク| *[MyVirtualNetwork]* を選択します。 |
    | Subnet |  *[mySubnet]* を選択します。 |
    |**プライベート DNS 統合**||
    |プライベート DNS ゾーンとの統合 | **[はい]** を選択します。 |
    |プライベート DNS ゾーン |*[(New)privatelink.database.windows.net]* を選択します。 |
    |||

1.  **[確認および作成]** を選択します。  **[確認および作成]**   ページが表示され、Azure によって構成が検証されます。 
2. " **検証に成功しました**" というメッセージが表示されたら、 **[作成]** を選択します。 
 
## <a name="connect-to-a-vm-using-remote-desktop-rdp"></a>リモート デスクトップ (RDP) を使用して VM に接続


**myVm* を作成した後、次のようにインターネットからそれに接続します。 

1. ポータルの検索バーで、「*myVm*」と入力します。

1. **[接続]** を選択します。 **[接続]** ボタンを選択すると、 **[Connect to virtual machine]\(仮想マシンに接続する\)** が開きます。

1. **[RDP ファイルのダウンロード]** を選択します。 リモート デスクトップ プロトコル ( *.rdp*) ファイルが作成され、お使いのコンピューターにダウンロードされます。

1. ダウンロードした .rdp* ファイルを開きます。

    1. メッセージが表示されたら、 **[Connect]** を選択します。

    1. VM の作成時に指定したユーザー名とパスワードを入力します。

        > [!NOTE]
        > 場合によっては、 **[その他]**  >  **[別のアカウントを使用する]** を選択して、VM の作成時に入力した資格情報を指定する必要があります。

1. **[OK]** を選択します。

1. サインイン処理中に証明書の警告が表示される場合があります。 証明書の警告を受信する場合は、 **[はい]** または **[続行]** を選択します。

1. VM デスクトップが表示されたら最小化してローカル デスクトップに戻ります。  

## <a name="access-the-sql-database-server-privately-from-the-vm"></a>VM から SQL データベース サーバーにプライベートにアクセスする

1.  *myVM* のリモート デスクトップで、PowerShell を開きます。
2. 「 `nslookup myserver.database.windows.net`」と入力します。 

    次のようなメッセージが返されます。
    ```azurepowershell
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    myserver.privatelink.database.windows.net
    Address:  10.0.0.5
    Aliases:   myserver.database.windows.net
3. Install [SQL Server Management Studio](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms?view=sql-server-2017).

4. In **Connect to server**, enter or select this information:

    | Setting | Value |
    | ------- | ----- |
    | Server type| Select **Database Engine**.|
    | Server name| Select *myserver.database.windows.net* |
    | User name | Enter a password provided during the SQL server creation. |
    |Password |Enter a password provided during the SQL server creation. |
    |Remember password|Select **Yes**.|
    |||
1. Select **Connect**.
2. Browse databases from left menu.
3. (Optionally) Create or query information from mydatabase.
4. Close the remote desktop connection to *myVm*. 

## Clean up resources 
When you're done using the private endpoint, SQL server, and the VM, delete the resource group and all of the resources it contains: 
1. Enter *myResourceGroup* in the **Search** box at the top of the portal and select *myResourceGroup* from the search results. 
2. Select **Delete resource group**. 
3. Enter myResourceGroup for **TYPE THE RESOURCE GROUP NAME** and select **Delete**.

## Next steps

In this quickstart, you created a VM on a virtual network, a SQL database server, and a private endpoint for private access. You connected to one VM from the internet and securely communicated to the SQL database server using Private Link. To learn more about private endpoints, see [What is Azure private endpoint?](private-endpoint-overview.md).

