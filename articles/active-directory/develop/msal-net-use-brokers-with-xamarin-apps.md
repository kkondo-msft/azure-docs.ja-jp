---
title: Xamarin iOS および Android アプリケーションで Microsoft Authenticator または Microsoft Intune ポータル サイトを使用する | Azure
description: .NET 用の Azure AD Authentication ライブラリ (ADAL.NET) から .NET 用 Microsoft 認証ライブラリ (MSAL.NET) に Microsoft Authenticator を使用することができる Xamarin iOS アプリケーションの移行の方法について説明します
documentationcenter: dev-center-name
author: jmprieur
manager: CelesteDG
editor: ''
ms.service: active-directory
ms.subservice: develop
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 09/08/2019
ms.author: jmprieur
ms.reviewer: saeeda
ms.custom: aaddev
ms.collection: M365-identity-device-management
ms.openlocfilehash: e16ad801b3b691d942c2ba0dc723ba72e24cf4c0
ms.sourcegitcommit: 23389df08a9f4cab1f3bb0f474c0e5ba31923f12
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2019
ms.locfileid: "70875600"
---
# <a name="use-microsoft-authenticator-or-microsoft-intune-company-portal-on-xamarin-applications"></a>Xamarin アプリケーションで Microsoft Authenticator または Microsoft Intune ポータル サイトを使用する

Android および iOS では、Microsoft Authenticator または Microsoft Intune ポータル サイトのようなブローカー (Android のみ):

- シングル サインオン (SSO)。 ユーザーがアプリケーションごとにサインインする必要がなくなります。
- デバイスの識別。 ワークプレースにデバイスが参加したときに作成されたデバイス証明書にアクセスすることで行います。
- アプリケーション ID の検証。 アプリケーションはブローカーを呼び出してリダイレクト URL を渡し、それがブローカーによって検証されます。

これらの機能のいずれかを有効にするには、アプリケーション開発者は、`PublicClientApplicationBuilder.CreateApplication` メソッドを呼び出すときに `WithBroker()` パラメーターを使用する必要があります。 `.WithBroker()` は既定で true に設定されます。 開発者は、[iOS](#brokered-authentication-for-ios) または [Android](#brokered-authentication-for-android) アプリケーションの次の手順にも従う必要があります。

## <a name="brokered-authentication-for-ios"></a>iOS のブローカー認証

次の手順に従って、Xamarin.iOS アプリが [Microsoft Authenticator](https://itunes.apple.com/us/app/microsoft-authenticator/id983156458) アプリと通信できるようにします。

### <a name="step-1-enable-broker-support"></a>手順 1:ブローカーのサポートを有効にする
ブローカーのサポートは、PublicClientApplication ベースごとに有効になります。 既定では無効になっています。 PublicClientApplicationBuilder を使用して PublicClientApplication を作成する場合は、`WithBroker()` パラメーターを使用します (既定では true に設定されています)。

```CSharp
var app = PublicClientApplicationBuilder
                .Create(ClientId)
                .WithBroker()
                .WithReplyUri(redirectUriOnIos) // $"msauth.{Bundle.Id}://auth" (see step 6 below)
                .Build();
```

### <a name="step-2-update-appdelegate-to-handle-the-callback"></a>手順 2:コールバックを処理するように AppDelegate を更新する
MSAL.NET がブローカーを呼び出すと、ブローカーは `AppDelegate` クラスの `OpenUrl` メソッドを使用してアプリケーションにコールバックします。 MSAL はブローカーからの応答を待つため、アプリケーションが協力して MSAL.NET を再度呼び出す必要があります。 この連携を可能にするには、`AppDelegate.cs` ファイルを更新して次のメソッドをオーバーライドします。

```CSharp
public override bool OpenUrl(UIApplication app, NSUrl url, 
                             string sourceApplication,
                             NSObject annotation)
{
    if (AuthenticationContinuationHelper.IsBrokerResponse(sourceApplication))
    {
      AuthenticationContinuationHelper.SetBrokerContinuationEventArgs(url);
      return true;
    }
    
    else if (!AuthenticationContinuationHelper.SetAuthenticationContinuationEventArgs(url))
    {               
         return false;                
    }
    
    return true;     
}           
```

このメソッドは、アプリケーションが起動されるたびに呼び出され、ブローカーからの応答を処理して MSAL.NET が開始した認証プロセスを完了するために使用されます。

### <a name="step-3-set-a-uiviewcontroller"></a>手順 3:UIViewController() を設定する
まだ `AppDelegate.cs` にいる場合は、オブジェクト ウィンドウを設定する必要があります。 通常、Xamarin iOS では、オブジェクト ウィンドウを設定する必要はありませんが、ブローカーから応答を送受信するには、オブジェクト ウィンドウが必要です。 

これを行うには、次の 2 つの操作を行う必要があります。 
1) `AppDelegate.cs` で `App.RootViewController` を新しい `UIViewController()` に設定します。 これを割り当てることにより、UIViewController でのブローカーへの呼び出しが保証されます。 正しく設定されていないと、次のエラーが表示されることがあります。`"uiviewcontroller_required_for_ios_broker":"UIViewController is null, so MSAL.NET cannot invoke the iOS broker. See https://aka.ms/msal-net-ios-broker"`
2) AcquireTokenInteractive の呼び出しで、`.WithParentActivityOrWindow(App.RootViewController)` を使用し、使用するオブジェクト ウィンドウへの参照を渡します。

**例:**

`App.cs`で、次のように記述します。
```CSharp
   public static object RootViewController { get; set; }
```
`AppDelegate.cs`で、次のように記述します。
```CSharp
   LoadApplication(new App());
   App.RootViewController = new UIViewController();
```
AcquireToken の呼び出しで、次のように記述します。
```CSharp
result = await app.AcquireTokenInteractive(scopes)
             .WithParentActivityOrWindow(App.RootViewController)
             .ExecuteAsync();
```

### <a name="step-4-register-a-url-scheme"></a>手順 4:URL スキームを登録する
MSAL.NET は、URL を使用してブローカーを呼び出し、ブローカーの応答をアプリに返します。 ラウンド トリップを終了するには、`Info.plist` ファイルにアプリの URL スキームを登録する必要があります。

`CFBundleURLSchemes` 名には `msauth.` がプレフィックスとして、お使いの `CFBundleURLName` の前に含まれている必要があります。

`$"msauth.(BundleId)"`

**例:** 
`msauth.com.yourcompany.xforms`

> [!NOTE]
> これは、ブローカーから応答を受信したときにアプリを一意に識別するために使用される RedirectUri の一部になります。

```XML
 <key>CFBundleURLTypes</key>
    <array>
      <dict>
        <key>CFBundleTypeRole</key>
        <string>Editor</string>
        <key>CFBundleURLName</key>
        <string>com.yourcompany.xforms</string>
        <key>CFBundleURLSchemes</key>
        <array>
          <string>msauth.com.yourcompany.xforms</string>
        </array>
      </dict>
    </array>
```

### <a name="step-5-lsapplicationqueriesschemes"></a>手順 5:LSApplicationQueriesSchemes
MSAL は、`–canOpenURL:` を使用してブローカーがデバイスにインストールされているかどうかを確認します。 iOS 9 では、アプリケーションがクエリを実行できるスキームが Apple によってロックダウンされました。 

`Info.plist` ファイルの `LSApplicationQueriesSchemes` セクションに **`msauthv2`** を**追加**します。

```XML 
<key>LSApplicationQueriesSchemes</key>
    <array>
      <string>msauthv2</string>
    </array>
```

### <a name="step-6-register-your-redirecturi-in-the-application-portal"></a>手順 6:アプリケーション ポータルで RedirectUri を登録する
ブローカーを使用すると、redirectUri に追加の要件が追加されます。 redirectUri は次の形式にする _**必要**_ があります。
```CSharp
$"msauth.{BundleId}://auth"
```
**例:**
```CSharp
public static string redirectUriOnIos = "msauth.com.yourcompany.XForms://auth"; 
```
**RedirectUri が `Info.plist` ファイルに含まれている `CFBundleURLSchemes` 名と一致していることがわかります。**

### <a name="step-7-make-sure-the-redirect-uri-is-registered-with-your-app"></a>手順 7: リダイレクト URI がアプリに登録されていることを確認する

このリダイレクト URI は、アプリ登録ポータル (https://portal.azure.com) でアプリケーションの有効なリダイレクト URI として登録されている必要があります。 

ポータルには、バンドル ID からのブローカー応答 URI を計算するのに役立つ、新しいエクスペリエンス アプリ登録ポータルが用意されています。

1. アプリの登録で、 **[認証]** を選択し、 **[新しいエクスペリエンスを試す]** を選択します。
   ![新しいアプリの登録エクスペリエンスを試す](media/msal-net-use-brokers-with-xamarin-apps/60799285-2d031b00-a173-11e9-9d28-ac07a7ae894a.png)

2. **[プラットフォームの追加]** を選択します。
   ![[プラットフォームの追加]](media/msal-net-use-brokers-with-xamarin-apps/60799366-4c01ad00-a173-11e9-934f-f02e26c9429e.png)

3. プラットフォームの一覧がサポートされている場合は、 **[iOS]** を選択します。
   ![iOS の構成](media/msal-net-use-brokers-with-xamarin-apps/60799411-60de4080-a173-11e9-9dcc-d39a45826d42.png)

4. 要求されたらバンドル ID を入力し、 **[登録]** を押します。
   ![バンドル ID の入力](media/msal-net-use-brokers-with-xamarin-apps/60799477-7eaba580-a173-11e9-9f8b-431f5b09344e.png)

5. リダイレクト URI が自動的に計算されます。
   ![リダイレクト URI のコピー](media/msal-net-use-brokers-with-xamarin-apps/60799538-9e42ce00-a173-11e9-860a-015a1840fd19.png)

## <a name="brokered-authentication-for-android"></a>Android のブローカー認証

Android では、ブローカー サポートは利用できません

## <a name="next-steps"></a>次の手順

[MSAL.NET でのユニバーサル Windows プラットフォームに固有の考慮事項](msal-net-uwp-considerations.md)