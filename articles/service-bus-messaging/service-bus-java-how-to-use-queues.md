---
title: Java で Azure Service Bus キューを使用する
description: Azure での Service Bus キューの使用方法を学習します。 コード サンプルは Java で記述されています。
services: service-bus-messaging
documentationcenter: java
author: axisc
manager: timlt
editor: spelluru
ms.assetid: f701439c-553e-402c-94a7-64400f997d59
ms.service: service-bus-messaging
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: Java
ms.topic: article
ms.date: 04/10/2019
ms.author: aschhab
ms.custom: seo-java-july2019, seo-java-august2019
ms.openlocfilehash: 84f035a55256ab22c823b2aaed9c9c2400eebf5b
ms.sourcegitcommit: ee61ec9b09c8c87e7dfc72ef47175d934e6019cc
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 08/30/2019
ms.locfileid: "70173418"
---
# <a name="use-azure-service-bus-queues-with-java-to-send-and-receive-messages"></a>Java で Azure Service Bus キューを使用してメッセージを送受信する
[!INCLUDE [service-bus-selector-queues](../../includes/service-bus-selector-queues.md)]
このチュートリアルでは、Java アプリケーションを作成して、Service Bus キューとの間でメッセージを送受信する方法を学習します。 

> [!NOTE]
> GitHub の [azure-service-bus のリポジトリ](https://github.com/Azure/azure-service-bus/tree/master/samples/Java)には、Java のサンプルがあります。

## <a name="prerequisites"></a>前提条件
1. Azure サブスクリプション。 このチュートリアルを完了するには、Azure アカウントが必要です。 [MSDN のサブスクライバー特典](https://azure.microsoft.com/pricing/member-offers/credit-for-visual-studio-subscribers/?WT.mc_id=A85619ABF)を有効にするか、[無料アカウント](https://azure.microsoft.com/free/?WT.mc_id=A85619ABF)にサインアップしてください。
2. 使用するキューがない場合は、「[Azure portal を使用して Service Bus キューを作成する](service-bus-quickstart-portal.md)」の記事にある手順に従って、キューを作成します。
    1. Service Bus **キュー**の**概要**をお読みください。 
    2. Service Bus **名前空間**を作成します。 
    3. **接続文字列**を取得します。
    4. Service Bus **キュー**を作成します。
3. [Azure SDK for Java][Azure SDK for Java] をインストールします。 


## <a name="configure-your-application-to-use-service-bus"></a>Service Bus を使用するようにアプリケーションを構成する
このサンプルを作成する前に [Azure SDK for Java][Azure SDK for Java] がインストールされていることを確認してください。 Eclipse を使用している場合は、Azure SDK for Java が含まれている [Azure Toolkit for Eclipse][Azure Toolkit for Eclipse] をインストールできます。 これで **Microsoft Azure Libraries for Java** をプロジェクトに追加できます。

![ライブラリを追加できる [ライブラリ] タブのスクリーンショット](./media/service-bus-java-how-to-use-queues/eclipselibs.png)

次の `import` ステートメントを Java ファイルの先頭に追加します。

```java
// Include the following imports to use Service Bus APIs
import com.google.gson.reflect.TypeToken;
import com.microsoft.azure.servicebus.*;
import com.microsoft.azure.servicebus.primitives.ConnectionStringBuilder;
import com.google.gson.Gson;

import static java.nio.charset.StandardCharsets.*;

import java.time.Duration;
import java.util.*;
import java.util.concurrent.*;

import org.apache.commons.cli.*;

```

## <a name="send-messages-to-a-queue"></a>メッセージをキューに送信する
Service Bus キューにメッセージを送信する場合、アプリケーションによって **QueueClient** オブジェクトがインスタンス化され、非同期的にメッセージが送信されます。 次のコードでは、ポータルを使用して作成されたキューにメッセージを送信する方法を示しています。

```java
public void run() throws Exception {
    // Create a QueueClient instance and then asynchronously send messages.
    // Close the sender once the send operation is complete.
    QueueClient sendClient = new QueueClient(new ConnectionStringBuilder(ConnectionString, QueueName), ReceiveMode.PEEKLOCK);
    this.sendMessageAsync(sendClient).thenRunAsync(() -> sendClient.closeAsync());

    sendClient.close();
}

    CompletableFuture<Void> sendMessagesAsync(QueueClient sendClient) {
        List<HashMap<String, String>> data =
                GSON.fromJson(
                        "[" +
                                "{'name' = 'Einstein', 'firstName' = 'Albert'}," +
                                "{'name' = 'Heisenberg', 'firstName' = 'Werner'}," +
                                "{'name' = 'Curie', 'firstName' = 'Marie'}," +
                                "{'name' = 'Hawking', 'firstName' = 'Steven'}," +
                                "{'name' = 'Newton', 'firstName' = 'Isaac'}," +
                                "{'name' = 'Bohr', 'firstName' = 'Niels'}," +
                                "{'name' = 'Faraday', 'firstName' = 'Michael'}," +
                                "{'name' = 'Galilei', 'firstName' = 'Galileo'}," +
                                "{'name' = 'Kepler', 'firstName' = 'Johannes'}," +
                                "{'name' = 'Kopernikus', 'firstName' = 'Nikolaus'}" +
                                "]",
                        new TypeToken<List<HashMap<String, String>>>() {}.getType());

        List<CompletableFuture> tasks = new ArrayList<>();
        for (int i = 0; i < data.size(); i++) {
            final String messageId = Integer.toString(i);
            Message message = new Message(GSON.toJson(data.get(i), Map.class).getBytes(UTF_8));
            message.setContentType("application/json");
            message.setLabel("Scientist");
            message.setMessageId(messageId);
            message.setTimeToLive(Duration.ofMinutes(2));
            System.out.printf("\nMessage sending: Id = %s", message.getMessageId());
            tasks.add(
                    sendClient.sendAsync(message).thenRunAsync(() -> {
                        System.out.printf("\n\tMessage acknowledged: Id = %s", message.getMessageId());
                    }));
        }
        return CompletableFuture.allOf(tasks.toArray(new CompletableFuture<?>[tasks.size()]));
    }

```

Service Bus キューに送信されたメッセージ (および Service Bus キューから受信したメッセージ) は、[Message](/java/api/com.microsoft.azure.servicebus.message?view=azure-java-stable) クラスのインスタンスになります。 Message オブジェクトには、(Label、TimeToLive などの) 標準的なプロパティ、アプリケーションに特有のカスタム プロパティの保持に使用するディクショナリ、任意のアプリケーション データの本体が備わっています。 アプリケーションでは、Message のコンストラクターにシリアル化可能なオブジェクトを渡すことによってメッセージの本文を設定できます。その後で、適切なシリアライザーを使用してオブジェクトをシリアル化します。 または、**java.IO.InputStream** オブジェクトを提供することもできます。


Service Bus キューでサポートされているメッセージの最大サイズは、[Standard レベル](service-bus-premium-messaging.md)では 256 KB、[Premium レベル](service-bus-premium-messaging.md)では 1 MB です。 標準とカスタムのアプリケーション プロパティが含まれるヘッダーの最大サイズは 64 KB です。 キューで保持されるメッセージ数には上限がありませんが、キュー 1 つあたりが保持できるメッセージの合計サイズには上限があります。 このキュー サイズは作成時に定義され、上限は 5 GB です。

## <a name="receive-messages-from-a-queue"></a>キューからメッセージを受信する
キューからメッセージを受信する主な方法は、**ServiceBusContract** オブジェクトを使用することです。 メッセージは 2 つの異なるモードで受信できます。**ReceiveAndDelete** と **PeekLock** です。

**ReceiveAndDelete** モードを使用する場合、受信が 1 回ずつの動作になります。つまり、Service Bus はキュー内のメッセージに対する読み取り要求を受け取ると、メッセージを読み取り中としてマークし、アプリケーションに返します。 **ReceiveAndDelete** モード (既定) は最もシンプルなモデルであり、障害発生時にアプリケーション側でメッセージを処理しないことを許容できるシナリオに最適です。 このことを理解するために、コンシューマーが受信要求を発行した後で、メッセージを処理する前にクラッシュしたというシナリオを考えてみましょう。
Service Bus はメッセージを読み取り済みとしてマークしているため、アプリケーションが再起動してメッセージの読み取りを再開すると、クラッシュ前に読み取られていたメッセージは見落とされます。

**PeekLock** モードでは、メッセージの受信処理が 2 段階の動作になり、メッセージが失われることが許容できないアプリケーションに対応することができます。 Service Bus は要求を受け取ると、次に読み取られるメッセージを検索して、他のコンシューマーが受信できないようロックしてから、アプリケーションにメッセージを返します。 アプリケーションがメッセージの処理を終えた後 (または後で処理するために確実に保存した後)、受信したメッセージに対して **Delete** を呼び出して受信処理の第 2 段階を完了します。 Service Bus が **Delete** の呼び出しを確認すると、メッセージが読み取り中としてマークされ、キューから削除されます。

次の例では、(既定モードではなく) **PeekLock** モードを使用したメッセージの受信および処理の方法を示しています。 次の例では、無限ループを使用して、`TestQueue` にメッセージが到着するごとに処理しています。

```java
    public void run() throws Exception {
        // Create a QueueClient instance for receiving using the connection string builder
        // We set the receive mode to "PeekLock", meaning the message is delivered
        // under a lock and must be acknowledged ("completed") to be removed from the queue
        QueueClient receiveClient = new QueueClient(new ConnectionStringBuilder(ConnectionString, QueueName), ReceiveMode.PEEKLOCK);
        this.registerReceiver(receiveClient);

        // shut down receiver to close the receive loop
        receiveClient.close();
    }
    void registerReceiver(QueueClient queueClient) throws Exception {
        // register the RegisterMessageHandler callback
        queueClient.registerMessageHandler(new IMessageHandler() {
        // callback invoked when the message handler loop has obtained a message
            public CompletableFuture<Void> onMessageAsync(IMessage message) {
            // receives message is passed to callback
                if (message.getLabel() != null &&
                    message.getContentType() != null &&
                    message.getLabel().contentEquals("Scientist") &&
                    message.getContentType().contentEquals("application/json")) {

                        byte[] body = message.getBody();
                        Map scientist = GSON.fromJson(new String(body, UTF_8), Map.class);

                        System.out.printf(
                            "\n\t\t\t\tMessage received: \n\t\t\t\t\t\tMessageId = %s, \n\t\t\t\t\t\tSequenceNumber = %s, \n\t\t\t\t\t\tEnqueuedTimeUtc = %s," +
                            "\n\t\t\t\t\t\tExpiresAtUtc = %s, \n\t\t\t\t\t\tContentType = \"%s\",  \n\t\t\t\t\t\tContent: [ firstName = %s, name = %s ]\n",
                            message.getMessageId(),
                            message.getSequenceNumber(),
                            message.getEnqueuedTimeUtc(),
                            message.getExpiresAtUtc(),
                            message.getContentType(),
                            scientist != null ? scientist.get("firstName") : "",
                            scientist != null ? scientist.get("name") : "");
                    }
                    return CompletableFuture.completedFuture(null);
                }

                // callback invoked when the message handler has an exception to report
                public void notifyException(Throwable throwable, ExceptionPhase exceptionPhase) {
                    System.out.printf(exceptionPhase + "-" + throwable.getMessage());
                }
        },
        // 1 concurrent call, messages are auto-completed, auto-renew duration
        new MessageHandlerOptions(1, true, Duration.ofMinutes(1)));
    }

```

## <a name="how-to-handle-application-crashes-and-unreadable-messages"></a>アプリケーションのクラッシュと読み取り不能のメッセージを処理する方法
Service Bus には、アプリケーションにエラーが発生した場合や、メッセージの処理に問題がある場合に復旧を支援する機能が備わっています。 受信側のアプリケーションが何らかの理由によってメッセージを処理できない場合には、受信したメッセージについて (**deleteMessage** メソッドの代わりに) **unlockMessage** メソッドを呼び出すことができます。 このメソッドが呼び出されると、Service Bus によってキュー内のメッセージのロックが解除され、メッセージが再度受信できる状態に変わります。メッセージを受信するアプリケーションは、以前と同じものでも、別のものでもかまいません。

キュー内でロックされているメッセージにはタイムアウトも設定されています。アプリケーションがクラッシュした場合など、ロックがタイムアウトになる前にアプリケーションがメッセージの処理に失敗した場合には、メッセージのロックが自動的に解除され、再度受信できる状態に変わります。

メッセージが処理された後、**deleteMessage** 要求が発行される前にアプリケーションがクラッシュした場合は、アプリケーションが再起動する際にメッセージが再配信されます。 一般的に、この動作は "*1 回以上の処理*" と呼ばれます。つまり、すべてのメッセージが 1 回以上処理されますが、特定の状況では、同じメッセージが再配信される可能性があります。 重複処理が許されないシナリオの場合、重複メッセージの配信を扱うロジックをアプリケーションに追加する必要があります。 通常、この問題はメッセージの **getMessageId** メソッドを使用して対処します。このプロパティは、配信が試行された後も同じ値を保持します。

> [!NOTE]
> Service Bus リソースは、[Service Bus Explorer](https://github.com/paolosalvatori/ServiceBusExplorer/) で管理できます。 Service Bus Explorer を使用すると、ユーザーは Service Bus 名前空間に接続し、簡単な方法でメッセージング エンティティを管理できます。 このツールには、インポート/エクスポート機能や、トピック、キュー、サブスクリプション、リレー サービス、通知ハブ、イベント ハブをテストする機能などの高度な機能が用意されています。 

## <a name="next-steps"></a>次の手順
これで、Service Bus キューの基本を学習できました。詳細については、「[Service Bus のキュー、トピック、サブスクリプション][Queues, topics, and subscriptions]」をご覧ください。

詳細については、 [Java デベロッパー センター](https://azure.microsoft.com/develop/java/)を参照してください。

[Azure SDK for Java]: https://docs.microsoft.com/java/api/overview/azure/
[Azure Toolkit for Eclipse]: https://docs.microsoft.com/java/azure/eclipse/azure-toolkit-for-eclipse
[Queues, topics, and subscriptions]: service-bus-queues-topics-subscriptions.md
[BrokeredMessage]: /dotnet/api/microsoft.servicebus.messaging.brokeredmessage
