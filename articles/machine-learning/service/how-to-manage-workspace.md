---
title: ポータルで Azure ML ワークスペースを作成する
titleSuffix: Azure Machine Learning service
description: Azure portal 内で Azure Machine Learning Services ワークスペースを作成、表示、削除する方法について説明します。
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.reviewer: jmartens
ms.author: shipatel
author: shivp950
ms.date: 05/10/2019
ms.custom: seodec18
ms.openlocfilehash: df015df3c20cdb99512167e3dc3e455da2762c52
ms.sourcegitcommit: 86d49daccdab383331fc4072b2b761876b73510e
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/06/2019
ms.locfileid: "70744779"
---
# <a name="create-and-manage-azure-machine-learning-service-workspaces-in-the-azure-portal"></a>Azure portal 内で Azure Machine Learning Services ワークスペースを作成および管理する

この記事では、[Azure Machine Learning Services](overview-what-is-azure-ml.md) 用の Azure portal 内で [**Azure Machine Learning Services ワークスペース**](concept-workspace.md)を作成、表示、削除します。  ワークスペースを初めて利用する際にはポータルが最も手軽ですが、ニーズに変化が生じたり自動化の要件が高まったりした場合は、[CLI](reference-azure-machine-learning-cli.md)、[Python コード](https://docs.microsoft.com/python/api/overview/azure/ml/intro?view=azure-ml-py)、または [VS Code 拡張機能](how-to-vscode-tools.md#get-started-with-azure-machine-learning)を使用して、ワークスペースを作成したり、削除したりすることもできます。

## <a name="create-a-workspace"></a>ワークスペースの作成

ワークスペースを作成するには、Azure サブスクリプションが必要です。 Azure サブスクリプションをお持ちでない場合は、開始する前に無料アカウントを作成してください。 [無料版または有料版の Azure Machine Learning service](https://aka.ms/AMLFree) を今日からお試しいただけます。

[!INCLUDE [aml-create-portal](../../../includes/aml-create-in-portal.md)]

### <a name="download-a-configuration-file"></a>構成ファイルをダウンロードする

1. [Notebook VM](tutorial-1st-experiment-sdk-setup.md#azure) を作成する場合は、この手順をスキップしてください。

1. このワークスペースを参照するローカル環境でコードを使用する場合は、ワークスペースの **[概要]** セクションから **[config. json をダウンロード]** を選択します。  

   ![config.json をダウンロードする](./media/how-to-manage-workspace/configure.png)
   
   このファイルは、Python スクリプトまたは Jupyter Notebook を含むディレクトリ構造内に置きます。 それは、同じディレクトリ内、 *.azureml* という名前のサブディレクトリ内、または親ディレクトリ内に置くことができます。 Notebook VM を作成するとき、このファイルは VM 上の正しいディレクトリに自動的に追加されます。


## <a name="view"></a>ワークスペースの表示

1. ポータルの左上隅にある **[すべてのサービス]** を選択します。

1. **[すべてのサービス]** フィルター フィールドに、「**Machine Learning service**」と入力します。  

1. **[Machine Learning service ワークスペース]** を選択します。

   ![Azure Machine Learning service ワークスペースを検索する](media/how-to-manage-workspace/all-services.png)

1. 見つかったワークスペースの一覧にひととおり確認します。 サブスクリプション、リソース グループ、場所に基づいてフィルター処理できます。  

1. ワークスペースを選択して、そのプロパティを表示します。
   ![ワークスペースのプロパティ](media/how-to-manage-workspace/allservices_view_workspace_full.PNG)

## <a name="delete-a-workspace"></a>ワークスペースを削除する

削除するワークスペースの上にある [削除] ボタンを使用します。

  ![[削除] ボタン](media/how-to-manage-workspace/delete-workspace.png)

## <a name="clean-up-resources"></a>リソースのクリーンアップ

[!INCLUDE [aml-delete-resource-group](../../../includes/aml-delete-resource-group.md)]

## <a name="next-steps"></a>次の手順

ワークスペースを使用し、Azure Machine Learning サービスでモデルをビルド、トレーニング、デプロイする方法については、完全版のチュートリアルをご利用ください。

> [!div class="nextstepaction"]
> [チュートリアル:モデルをトレーニングする](tutorial-train-models-with-aml.md)
