---
title: Form Recognizer 向けコンテナーのインストールと実行
titleSuffix: Azure Cognitive Services
description: Form Recognizer コンテナーを使用してフォームとテーブルのデータを解析する方法を学習します。
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: conceptual
ms.date: 08/29/2019
ms.author: dapine
ms.openlocfilehash: 25ea4c96a0e392db2af9c25a150696ca2b25b2dd
ms.sourcegitcommit: 19a821fc95da830437873d9d8e6626ffc5e0e9d6
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/29/2019
ms.locfileid: "70164552"
---
# <a name="install-and-run-form-recognizer-containers"></a>Form Recognizer コンテナーのインストールと実行

Azure Form Recognizer では、機械学習テクノロジを適用して、フォームにあるキーと値のペアおよびテーブルを識別して抽出します。 値とテーブル エントリをキー値のペアに関連付けてから、元のファイル内の関係を含む構造化データを出力します。 

複雑さを軽減し、ワークフロー自動化プロセスまたは他のアプリケーションにカスタム Form Recognizer モデルを簡単に統合するために、単純な REST API を使用してモデルを呼び出すことができます。 必要なのは 5 つの形式のドキュメント (または 1 つの空のフォームと 2 つの入力済みフォーム) だけなので、すばやく正確に、特定のコンテンツに合わせて調整された結果を取得できます。 手動での大掛かりな介入や広範なデータ サイエンスの専門知識は必要ありません。 また、データのラベル付けやデータの注釈付けは必要ありません。

|Function|機能|
|-|-|
|Form Recognizer| <li>PDF、PNG、JPG ファイルを処理する<li>同じレイアウトのフォームを少なくとも 5 つ使用してカスタム モデルをトレーニングする <li>キーと値のペアおよびテーブル情報を抽出する <li>Azure Cognitive Service の Computer Vision API のテキスト認識機能を使用して、フォーム内の画像から印刷されたテキストを検出し、抽出する<li>注釈またはラベル付けを必要としない|

Azure サブスクリプションをお持ちでない場合は、開始する前に [無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) を作成してください。

## <a name="prerequisites"></a>前提条件

Form Recognizer コンテナーを使用する前に、次の前提条件を満たす必要があります。

|必須|目的|
|--|--|
|Docker エンジン| [ホスト コンピューター](#the-host-computer)に Docker エンジンをインストールしておく必要があります。 Docker には、[macOS](https://docs.docker.com/docker-for-mac/)、[Windows](https://docs.docker.com/docker-for-windows/)、[Linux](https://docs.docker.com/engine/installation/#supported-platforms) 上で Docker 環境の構成を行うパッケージが用意されています。 Docker やコンテナーの基礎に関する入門情報については、「[Docker overview](https://docs.docker.com/engine/docker-overview/)」(Docker の概要) を参照してください。<br><br> コンテナーが Azure に接続して課金データを送信できるように、Docker を構成する必要があります。 <br><br> Windows では、Linux コンテナーをサポートするように Docker を構成することも必要です。<br><br>|
|Docker に関する知識 | レジストリ、リポジトリ、コンテナー、コンテナー イメージなど、Docker の概念の基本的な理解に加えて、基本的な `docker` コマンドの知識が必要です。|
|Azure CLI| [Azure CLI](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) をホストにインストールします。|
|Computer Vision API リソース| スキャンしたドキュメントおよび画像を処理するには、Computer Vision リソースが必要です。 Azure リソース (REST API または SDK) または *cognitive-services-recognize-text* [コンテナー](../Computer-vision/computer-vision-how-to-install-containers.md##get-the-container-image-with-docker-pull)のいずれかとしてテキスト認識機能にアクセスできます。 通常の課金の料金が適用されます。 <br><br>Computer Vision リソース (Azure クラウドまたは Cognitive Services コンテナー) の API キーとエンドポイントの両方を渡します。 この API キーとエンドポイントを **{COMPUTER_VISION_API_KEY}** と **{COMPUTER_VISION_ENDPOINT_URI}** として使用します。<br><br> *cognitive-services-recognize-text* コンテナーを使用する場合は、以下のことを確認します。<br><br>Form Recognizer コンテナーの Computer Vision キーは、*cognitive-services-recognize-text* コンテナーの Computer Vision `docker run` コマンドで指定されたキーです。<br>課金エンドポイントは、コンテナーのエンドポイントです (`http://localhost:5000` など)。 同じホスト上で Computer Vision コンテナーと Form Recognizer コンテナーの両方を一緒に使用する場合、既定のポート *5000* を使用してその両方を起動することはできません。  |
|Form Recognizer リソース |これらのコンテナーを使用するためには、以下が必要です。<br><br>関連付けられている API キーとエンドポイント URI を取得するための Azure **Form Recognizer** リソース。 どちらの値も、Azure portal の **Form Recognizer** の [概要] ページと [キー] ページで入手でき、コンテナーを起動するには両方の値が必要です。<br><br>**{FORM_RECOGNIZER_API_KEY}** :[キー] ページにある 2 つのリソース キーのうちのどちらか<br><br>**{FORM_RECOGNIZER_ENDPOINT_URI}** :[概要] ページで提供されるエンドポイント|

## <a name="request-access-to-the-container-registry"></a>コンテナー レジストリへのアクセスの要求

コンテナーへのアクセスを要求するには、最初に [Cognitive Services Form Recognizer コンテナー アクセス要求フォーム](https://aka.ms/FormRecognizerRequestAccess)に記入して送信する必要があります。 そうすることで、Computer Vision にサインアップすることにもなります。 Computer Vision 要求フォームに別途サインアップする必要はありません。 

[!INCLUDE [Request access to the container registry](../../../includes/cognitive-services-containers-request-access-only.md)]

[!INCLUDE [Authenticate to the container registry](../../../includes/cognitive-services-containers-access-registry.md)]

## <a name="the-host-computer"></a>ホスト コンピューター

[!INCLUDE [Host Computer requirements](../../../includes/cognitive-services-containers-host-computer.md)]

### <a name="container-requirements-and-recommendations"></a>コンテナーの要件と推奨事項

次の表に、各 Form Recognizer コンテナーに割り当てる CPU コアとメモリの最小値と推奨値を示します。

| コンテナー | 最小値 | 推奨 |
|-----------|---------|-------------|
| Form Recognizer | 2 コア、4 GB メモリ | 4 コア、8 GB メモリ |
| テキスト認識 | 1 コア、8 GB のメモリ | 2 コア、8 GB のメモリ |

* 各コアは少なくとも 2.6 ギガヘルツ (GHz) 以上にする必要があります。
* コアとメモリは、`docker run` コマンドの一部として使用される `--cpus` と `--memory` の設定に対応します。

> [!Note]
> 最小値と推奨値は、Docker の制限に基づくもので、ホスト マシンのリソースに基づくものでは*ありません*。

## <a name="get-the-container-images-with-the-docker-pull-command"></a>docker pull コマンドでコンテナー イメージを入手する

**Form Recognizer** と**テキスト認識**の両オファリングのコンテナー イメージは、次のコンテナー レジストリにあります。

| コンテナー | イメージの完全修飾名 |
|-----------|------------|
| Form Recognizer | `containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest` |
| テキスト認識 | `containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest` |

両方のコンテナーが必要です。**テキスト認識**コンテナーの詳細については、[この記事以外で説明しています。](../Computer-vision/computer-vision-how-to-install-containers.md##get-the-container-image-with-docker-pull)

[!INCLUDE [Tip for using docker list](../../../includes/cognitive-services-containers-docker-list-tip.md)]

### <a name="docker-pull-for-the-form-recognizer-container"></a>Form Recognizer コンテナー用の docker pull

#### <a name="form-recognizer"></a>Form Recognizer

Form Recognizer コンテナーを入手するには、次のコマンドを使用します。

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer:latest
```
### <a name="docker-pull-for-the-recognize-text-container"></a>テキスト認識コンテナー用の Docker pull

#### <a name="recognize-text"></a>テキスト認識

テキスト認識コンテナーを入手するには、次のコマンドを使用します。

```Docker
docker pull containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text:latest
```

## <a name="how-to-use-the-container"></a>コンテナーを使用する方法

コンテナーを[ホスト コンピューター](#the-host-computer)上に用意できたら、次の手順を使用してコンテナーを操作します。

1. 必要な課金設定を使用して[コンテナーを実行](#run-the-container-by-using-the-docker-run-command)します。 `docker run` コマンドの他の[例](form-recognizer-container-configuration.md#example-docker-run-commands)もご覧いただけます。
1. [コンテナーの予測エンドポイントに対するクエリを実行します](#query-the-containers-prediction-endpoint)。

## <a name="run-the-container-by-using-the-docker-run-command"></a>docker run コマンドを使用してコンテナーを実行する

3 つのコンテナーのいずれかを実行するには、[docker run](https://docs.docker.com/engine/reference/commandline/run/) コマンドを使用します。 このコマンドには、次のパラメーターが使用されます。

| プレースホルダー | 値 |
|-------------|-------|
|{FORM_RECOGNIZER_API_KEY} | このキーは、コンテナーを起動するために使用されます。 Azure portal の **Form Recognizer の [キー]** ページで入手できます。  |
|{FORM_RECOGNIZER_ENDPOINT_URI} | 課金エンドポイント URI の値は、Azure portal の **Form Recognizer の [概要]** ページで入手できます。|
|{COMPUTER_VISION_API_KEY}| このキーは、Azure portal の **Computer Vision API の [キー]** ページで入手できます。|
|{COMPUTER_VISION_ENDPOINT_URI}|課金エンドポイント。 クラウドベースの Computer Vision リソースを使用している場合、URI 値は Azure portal の **Computer Vision API の [概要]** ページで入手できます。 `cognitive-services-recognize-text` コンテナーを使用している場合は、`docker run` コマンドでコンテナーに渡された課金エンドポイント URL を使用します。|

次の例の `docker run` コマンドでは、これらのパラメーターをお客様独自の値に置き換えてください。

### <a name="form-recognizer"></a>Form Recognizer

```bash
docker run --rm -it -p 5000:5000 --memory 8g --cpus 2 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY} \
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

このコマンドは、次の操作を行います。

* コンテナー イメージから Form Recognizer コンテナーを実行します。
* 2 つの CPU コアと 8 ギガバイト (GB) のメモリを割り当てます。
* TCP ポート 5000 を公開し、コンテナーに pseudo-TTY を割り当てます。
* コンテナーの終了後にそれを自動的に削除します。 ホスト コンピューター上のコンテナー イメージは引き続き利用できます。
* /input と /output のボリュームをコンテナーにマウントします。

[!INCLUDE [Running multiple containers on the same host H2](../../../includes/cognitive-services-containers-run-multiple-same-host.md)]

### <a name="run-separate-containers-as-separate-docker-run-commands"></a>別個のコンテナーを別個の docker run コマンドとして実行する

同じホスト上でローカルにホストされている Form Recognizer とテキスト認識エンジンの組み合わせに対して、以下に示す 2 つの Docker CLI コマンドの例を実行します。

ポート 5000 上で最初のコンテナーを実行します。 

```bash 
docker run --rm -it -p 5000:5000 --memory 4g --cpus 1 \
--mount type=bind,source=c:\input,target=/input  \
--mount type=bind,source=c:\output,target=/output \
containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer \
Eula=accept \
Billing={FORM_RECOGNIZER_ENDPOINT_URI} \
ApiKey={FORM_RECOGNIZER_API_KEY}
FormRecognizer:ComputerVisionApiKey={COMPUTER_VISION_API_KEY} \
FormRecognizer:ComputerVisionEndpointUri={COMPUTER_VISION_ENDPOINT_URI}
```

ポート 5001 上で 2 番目のコンテナーを実行します。

```bash 
docker run --rm -it -p 5001:5000 --memory 4g --cpus 1 \
containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text \
Eula=accept \
Billing={COMPUTER_VISION_ENDPOINT_URI} \
ApiKey={COMPUTER_VISION_API_KEY}
```
後続の各コンテナーは、別のポート上になっている必要があります。 

### <a name="run-separate-containers-with-docker-compose"></a>Docker Compose を使用して別個のコンテナーを実行する

同じホスト上でローカルにホストされている Form Recognizer とテキスト認識エンジンの組み合わせについて、Docker Compose YAML ファイルの次の例を参照してください。 テキスト認識エンジンの `{COMPUTER_VISION_API_KEY}` は、`formrecognizer` と `ocr` の両方のコンテナーで同じでなければなりません。 `formrecognizer` コンテナーでは `ocr` の名前とポートを使用するため、`{COMPUTER_VISION_ENDPOINT_URI}` は `ocr` コンテナーでのみ使用されます。 

```docker
version: '3.3'
services:   
  ocr:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-recognize-text"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{COMPUTER_VISION_ENDPOINT_URI}"
      apikey: "{COMPUTER_VISION_API_KEY}"

  formrecognizer:
    image: "containerpreview.azurecr.io/microsoft/cognitive-services-form-recognizer"
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 8g
        reservations:
          cpus: '1'
          memory: 4g
    environment:
      eula: accept
      billing: "{FORM_RECOGNIZER_ENDPOINT_URI}"
      apikey: "{FORM_RECOGNIZER_API_KEY}"
      FormRecognizer__ComputerVisionApiKey: {COMPUTER_VISION_API_KEY}
      FormRecognizer__ComputerVisionEndpointUri: "http://ocr:5000"
      FormRecognizer__SyncProcessTaskCancelLimitInSecs: 75
    links:
      - ocr
    volumes:
      - type: bind
        source: c:\output
        target: /output
      - type: bind
        source: c:\input
        target: /input
    ports:
      - "5000:5000"
```

> [!IMPORTANT]
> コンテナーを実行するには、`Eula`、`Billing`、`ApiKey` に加えて `FormRecognizer:ComputerVisionApiKey` と `FormRecognizer:ComputerVisionEndpointUri` のオプションを指定する必要があります。そうしないと、コンテナーが起動しません。 詳細については、「[課金](#billing)」を参照してください。

## <a name="query-the-containers-prediction-endpoint"></a>コンテナーの予測エンドポイントに対するクエリの実行

|コンテナー|エンドポイント|
|--|--|
|form-recognizer|http://localhost:5000

### <a name="form-recognizer"></a>Form Recognizer

コンテナーは、websocket ベースのクエリ エンドポイント API シリーズを提供します。これには、[Form Recognizer サービス SDK のドキュメント](https://docs.microsoft.com/azure/cognitive-services/form-recognizer/)を介してアクセスします。

Form Recognizer SDK では、既定でオンライン サービスを使用します。 コンテナーを使用するには、初期化方法を変更する必要があります。 次の例を参照してください。

#### <a name="for-c"></a>C# の場合

この Azure クラウド初期化呼び出しを使用する方法を

```csharp
var config =
    FormRecognizerConfig.FromSubscription(
        "YourSubscriptionKey",
        "YourServiceRegion");
```
次のようにコンテナー エンドポイントを使用する呼び出しに変更します。

```csharp
var config =
    FormRecognizerConfig.FromEndpoint(
        "ws://localhost:5000/formrecognizer/v1.0-preview/custom",
        "YourSubscriptionKey");
```

#### <a name="for-python"></a>Python の場合

この Azure クラウド初期化呼び出しを使用する方法を

```python
formrecognizer_config =
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key, region=service_region)
```

次のようにコンテナー エンドポイントを使用する呼び出しに変更します。

```python
formrecognizer_config = 
    formrecognizersdk.FormRecognizerConfig(
        subscription=formrecognizer_key,
        endpoint="ws://localhost:5000/formrecognizer/v1.0-preview/custom"
```

### <a name="form-recognizer"></a>Form Recognizer

コンテナーは、REST エンドポイント API を提供します。これは、[Form Recognizer API](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api/operations/AnalyzeWithCustomModel) のページにあります。


[!INCLUDE [Validate container is running - Container's API documentation](../../../includes/cognitive-services-containers-api-documentation.md)]


## <a name="stop-the-container"></a>コンテナーの停止

[!INCLUDE [How to stop the container](../../../includes/cognitive-services-containers-stop.md)]

## <a name="troubleshooting"></a>トラブルシューティング

コンテナーを実行すると、コンテナーは **stdout** と **stderr** を使用して、コンテナーの起動時や実行時に発生する問題のトラブルシューティングに役立つ情報を出力します。

## <a name="billing"></a>課金

Form Recognizer コンテナーでは、Azure アカウントの _Form Recognizer_ リソースを使用して、Azure に課金情報を送信します。

[!INCLUDE [Container's Billing Settings](../../../includes/cognitive-services-containers-how-to-billing-info.md)]

これらのオプションの詳細については、「[コンテナーの構成](form-recognizer-container-configuration.md)」を参照してください。

<!--blogs/samples/video courses -->

[!INCLUDE [Discoverability of more container information](../../../includes/cognitive-services-containers-discoverability.md)]

## <a name="summary"></a>まとめ

この記事では、Form Recognizer コンテナーの概念とそのダウンロード、インストール、および実行のワークフローについて学習しました。 要約すると:

* Form Recognizer は、Docker 用の Linux コンテナーを 1 つ提供します。
* コンテナー イメージは、Azure のプライベート コンテナー レジストリからダウンロードされます。
* コンテナー イメージを Docker で実行します。
* REST API または REST SDK のいずれかを使用して、コンテナーのホスト URI を指定することによって、Form Recognizer コンテナーの操作を呼び出すことができます。
* コンテナーをインスタンス化するときには、課金情報を指定する必要があります。

> [!IMPORTANT]
>  Cognitive Services コンテナーは、計測のために Azure に接続していないと、実行のライセンスが許可されません。 お客様は、コンテナーが常に計測サービスに課金情報を伝えられるようにする必要があります。 Cognitive Services コンテナーが、顧客データ (解析対象の画像やテキストなど) を Microsoft に送信することはありません。

## <a name="next-steps"></a>次の手順

* 構成設定について、[コンテナーの構成](form-recognizer-container-configuration.md)を確認します。
* さらに [Cognitive Services コンテナー](../cognitive-services-container-support.md)を使用します。
