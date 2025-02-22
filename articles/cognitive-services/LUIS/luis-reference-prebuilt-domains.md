---
title: 事前構築済みのドメインのリファレンス - LUIS
titleSuffix: Azure Cognitive Services
description: 事前構築済みのドメインのリファレンスです。事前構築済みのドメインは、Language Understanding Intelligent Service (LUIS) の意図とエンティティの事前構築済みのコレクションです。
services: cognitive-services
author: diberry
manager: nitinme
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 09/04/2019
ms.author: diberry
ms.openlocfilehash: b840f1ce42c9d7e4af8854a2c6bd7fd26f5b88e9
ms.sourcegitcommit: f176e5bb926476ec8f9e2a2829bda48d510fbed7
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/04/2019
ms.locfileid: "70307430"
---
# <a name="prebuilt-domain-reference-for-your-luis-app"></a>LUIS アプリの事前構築済みのドメインのリファレンス
このリファレンスは、[事前構築済みのドメイン](luis-how-to-use-prebuilt-domains.md)に関する情報を提供します。事前構築済みのドメインは、LUIS が提供している意図とエンティティの事前構築済みのコレクションです。

対照的に、[カスタム ドメイン](luis-how-to-start-new-app.md)は意図とモデルがない状態から始まります。 任意の事前構築済みのドメインの意図とエンティティをカスタム モデルに追加できます。

# <a name="supported-domains-across-cultures"></a>カルチャ全体でサポートされているドメイン

次の表は、現在サポートされているドメインをまとめたものです。 通常、他の言語より、英語のサポートが完成されています。 

| エンティティの種類       | EN-US      | ZH-CN   | DE    | FR     | ES    | IT      | PT-BR |  JP  |      KO |        NL |    TR |
|:-----------------:|:-------:|:-------:|:-----:|:------:|:-----:|:-------:| :-------:| :-------:| :-------:| :-------:|  :-------:| 
| [Calendar](#calendar)    | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [Communication](#communication)   | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [電子メール](#email)           | ✓    | ✓       | ✓   | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [HomeAutomation](#homeautomation)           | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [メモ](#notes)      | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [Places](#places)    | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [RestaurantReservation](#restaurantreservation)   | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [ToDo](#todo)     | ✓    | ✓       | ✓    | ✓     | ✓     | ✓  | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [Utilities](#utilities)          | ✓    | ✓        | ✓    | ✓      | ✓     | ✓       | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [Weather](#weather)        | ✓    | ✓        | ✓    | ✓      | ✓     | ✓       | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |
| [Web](#web)    | ✓    | ✓        | ✓    | ✓      | ✓     | ✓       | ✓  | ✓      | ✓    | ✓    | ✓     | ✓  |

事前構築済みドメインは次の場所では**サポートされていません**。

* フランス語 (カナダ)
* ヒンディー語
* スペイン語 (メキシコ)

# <a name="description-for-luis-prebuilt-domains"></a>LUIS 事前構築済みドメインの説明
## <a name="calendar"></a>**Calendar**
Calendar では、個人的な会合や予定に関することを扱います。パブリック イベント (ワールド カップのスケジュールやシアトルのイベント カレンダーなど)、一般的なカレンダー (曜日や立秋、労働者の日など) ではありません。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------------
 AcceptEventEntry | カレンダーで予定/会議/イベントを承諾します。 | 予定を承諾します。 <br> イベントを承諾します。 <br> 本日の会議を承諾します。
 Cancel | 仮想アシスタントで進行中のアクションを取り消します。たとえば、会議作成プロセスを取り消します。 <br> ***注意**:この意図では主に、"Cancel" アクションは Calendar シナリオに含まれます。 "Cancel" で汎用的な式が必要であれば、**Utilities** ドメインで "Cancel" 意図を活用してください。* | 問題ありません。イベントをキャンセルして。 <br> いいえ。予定をキャンセルして。
 ChangeCalendarEntry | 予定表のエントリを変更するか、再スケジュールします。 | 明日午前 6 時の予定を 午後 2 時に再スケジュールして。 <br> 午後 5 時に予定していた医師の予約を再スケジュールして。 <br> Jenny Olson との昼食を金曜日に再スケジュールして。 <br> イベント時間を変更して。
 CheckAvailability | ユーザーのカレンダーや他のユーザーのカレンダーで、予定や会議の空き状況を検索します。 | Jim はいつ会うことができる? <br> 明日、Carol が空いている時間を表示して。 <br> Chris は土曜日に空いている?
 Confirm | 前の意図に基づいて操作/アクションを実行するかどうかを確認します。 <br> ***注意**:この意図では主に、"Confirm" アクションは Calendar シナリオ向けとして含まれます。 "Confirm" でさらに一般的な表現が必要であれば、**Utilities** ドメインで "Confirm" 意図を活用してください。*| そのとおり。会議を作成して。 <br> はい、ありがとう。会議に接続して。
 ConnectToMeeting | 会議に接続します。 | Andy との 11 時の電話会議に接続して。 <br> 予算会議の呼び出しを承諾して。
 ContactMeetingAttendees | 会議出席者に連絡します。 | 3 時の会議に遅れると連絡して。 <br> 午前 8 時の会議を 8 時 30 分から始める必要があると同僚に通知して。
 CreateCalendarEntry | 新しい 1 回限りの項目をカレンダーに追加します。 | 問題について話し合うための会議を作成して。 <br> abc@microsoft.com との会議を作成して。
 DeleteCalendarEntry | カレンダー エントリを削除する要求。| Carol との約束をキャンセルして。 <br> 午前 9 時の 会議を削除して。 <br> イベントを削除して。
  FindCalendarEntry | カレンダー アプリケーションを開くか、カレンダーのエントリを検索します。 | 歯科の診察予定を探して。 <br> カレンダーを表示して。 <br> 木曜日の予定は?
 FindCalendarWhen | スケジュールが行われる時刻を確認します。 | Amber と Susan とはいつ会うの? <br> ブランチの予定はいつ? 
 FindCalendarWhere | スケジュールが行われる場所を確認します。 | 明日の約束の場所はどこ? <br>明日、Stacy と Michael とのディナーはどこ?
  FindCalendarWho | ターゲット スケジュールに参加する参加者を確認します。 | 明日 2 時の会議の確認されている出席者を教えて。 <br> 次の看護師会議に Jim は来るの?
 FindCalendarDetail | スケジュールの詳細を確認し、表示します。 | 同僚の Paul との間で予定している会議の詳細を教えて。
 FindDuration | 期間を確認します。 | 買い物を取りに行く時間はどのくらいあるの? <br> 昼食の時間はどのくらいあるの?
 FindMeetingRoom | 利用可能な会議室を検索します。 | どの会議室が使える? <br> 新しい会議の場所を 1 つ検索して。
 GoBack | 最後の手順または項目に戻ります。  <br> ***注意**:より汎用的な GoBack の発話については、**Utilities** ドメインを参照してください。* | 1 つ戻って。 <br> 最後のメールに戻って。
 Reject | ユーザーは仮想アシスタントの提案を拒否します。 <br> ***注意**:より汎用的な Reject の発話については、**Utilities** ドメインを参照してください。* | イベントを設定する必要はありません。 <br> その時間はやることが他にあります。
ShowNext | 次のイベントを確認します。 <br> ***注意**:より汎用的な ShowNext の発話については、**Utilities** ドメインを参照してください。* | 次のイベントは何? <br> 次の予定は何?
 ShowPrevious | 前のイベントを確認します。 <br> ***注意**:より汎用的な ShowPrevious の発話については、**Utilities** ドメインを参照してください。* | それの前のスケジュールは何?
 TimeRemaining | 次のイベントまでの残り時間を確認します。 | 会議までの残り時間を表示して。 <br> 次の会議が始まるまでの時間を表示して。
 
### <a name="entities"></a>**エンティティ**
エンティティ名 | エンティティの種類 | 説明 | 例 | スロット
-------|-----------------------|-------------|---------|--------
ContactName | personName | 連絡先または会議出席者の名前。 | **Betsy** に会う。 <br>  7 月 3 日午後 7 時に **Aubrey** に会う。 | Betsy <br> Aubrey <br> Amy 
DestinationCalendar | simple | ターゲット カレンダーの名前。 | 火曜日の 12 時にお母さんとランチ。**個人用** <br> **Google** カレンダーを既定のカレンダーとして使用します。 <br> ヨガ教室を月曜と水曜日の午後 3 時に変更して。 **個人用**の予定表の予定。 | Google <br> 個人用 <br> ビジネス <br> メイン
Duration | datetime | 会議の時間、予定の時間、残存時間。 | 明日の午前 11 時、奨学金の詳細について話し合うので Gary との会議を職場の予定表に追加して。時間は **20 分**。 <br> 金曜日、地下鉄でのイベントをカレンダーに追加して。午前 9 時に Thomas と食事。時間は **1 時間**。 | 1 時間 <br> 2 日 <br> 20 分 
EndDate | datetime | 会議または予定の終了日。 | カレンダーに 5 月 3 日から **5 月 5 日**までの Bass Hall でのコンサートを追加して | 5 月 5 日  
EndTime | datetime | 会議または予定の終了時刻。 | 2 時 30 分から **3 時**にできる? | three 
Location | simple | カレンダー項目、会議、予定の場所。  たとえば、住所、都市、地域などです。 | **Fremont** での会議を入れて。ミャンマーのタブレットについて。 <br> **Edina** で無償奉仕。 | 209 ナッシュビル ジム <br> 897 パンケーキ ハウス <br> ガレージ 
MeetingRoom | simple | 会議または予定に使われる部屋。 | 午後 2 時に Jake に会う。仕事の予定表に追加して。 場所は彼の**事務所**。今週の金曜日。 | 彼の事務所 <br> 会議室 <br> 第 2 ルーム
MoveEarlierTimeSpan | datetime | 会議または予定の時刻を早めるとき、その変更後の時刻。 | ランチ デートの時間を **30 分**前倒し。 | 30 分 <br> 2 時間 
MoveLaterTimeSpan |  datetime | 会議または予定の時刻を遅らせるとき、その変更後の時刻。 | Orchid Box との会議を **4 時間**遅らせて。 | 4 時間 <br> 約 15 分 
OrderReference | simple | 取得する項目を識別する、リスト内の順序または相対位置。 | 明日に入っている次の予定は何? | [次へ]
OriginalEndDate | datetime | 会議または予定のもともとの終了日。 | 休暇の終了日を**金曜日**から月曜に変更 | 金曜日 
OriginalEndTime | datetime | 会議または予定のもともとの終了時刻。 | **3 時**の終了予定を 4 時に | 3
OriginalStartDate | datetime | 会議または予定のもともとの開始日。 | **明日**午前 10 時の予定を 水曜の午前 9 時に変更して。  | 明日 
OriginalStartTime | datetime | 会議または予定のもともとの開始時刻。 | 明日**午前 10 時**の予定を 水曜の午前 9 時に変更して。 | 午前 10 時
PositionReference | ordinal | 取得する項目を識別する、リスト内の絶対位置。 | **2** つ目 | 秒 <br> いいえ。 3 <br> 番号 5
RelationshipName | list | 連絡先のリレーションシップ名。 | 午後 1 時にランチを追加。 **妻**と。 | 妻 <br> 夫 <br> 妹 
SlotAttribute | simple | ユーザーが問い合わせるか、編集する属性。 | イベントの**場所**を変更。 <br> **時刻**を 7 時に変更。 | location <br> time 
StartDate | datetime | 会議または予定の開始日。 | **水曜日**の午後 4 時に会議を作成して。 | 水曜日 
StartTime | datetime | 会議または予定の開始時刻。 | 水曜日の**午後 4 時**に会議を作成して。 | 午後 4 時
サブジェクト | simple、pattern.Any | 会議または予定のタイトルなどの、件名。 | **パーティーの準備**会議は何時? | 歯医者 <br> Julia とランチ 
Message | simple、pattern.Any | 出席者に送信するメッセージ。 | **私は遅れる**と夕食会の参加者に伝えて。 | 私は遅れます

## <a name="communication"></a>**Communication**
電話をかける、SMS/インスタント メッセージを送信する、連絡を検索して追加するなど、コミュニケーション (通常は発信) に関連するさまざまな要求を行います。 _連絡先の名前のみ_の発話は Communication ドメインに属しません。

### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
AddContact | ユーザーの連絡先リストに新しい連絡先を追加します。 | 新しい連絡先を追加して。  <br>   この番号を Jane という名前で保存して。
AddFlag | メールにフラグを追加します。 | このメールにフラグを付けて。 <br> このメールにフラグを追加して。
AddMore | 段階的な電子メールまたはテキストの作成の一部として、電子メールまたはテキストに追加します。 | テキストにさらに追加して。  <br>   メールの本文にさらに追加して。
Answer | 電話の着信に応答します。 | 電話に応答して。  <br>   電話に出て。
AssignContactNickname | 連絡先にニックネームを割り当てます。 | Isaac をお父さんに変更して。  <br>   Jim のニックネームを編集して。 <br>   Patti Owens にニックネームを追加して。
CallBack | 折り返し電話します。 | 折り返し電話して。
CallVoiceMail | ユーザーのボイス メールに接続します。 | ボイスメール ボックスに接続して。 <br>ボイス メールを送って。 <br>   ボイスメールを発信して。
Cancel | 操作を取り消します。 | キャンセルして。 <br>   終了して。
CheckIMStatus | IM で連絡先の状態を確認します。 | Jim のオンライン状態は退席中? 
CheckMessages | 新しいメッセージかメールがないか確認して。 | 受信トレイを確認して。 <br>  新しいメールは来ていない?
Confirm | アクションを確認します。 | はい、送信して。 <br> はい、このメールの送信を確定します。
EndCall | 電話を終了します。 | 電話を切って。 <br> 終了して。
FindContact | 連絡先情報を名前で検索します。 | ママの番号を見つけて。 <br>   Carol の番号を表示して。
FinishTask | 現在のタスクを完了。 | 終わりました。 <br> これで全部。
TurnForwardingOff | 着信の転送をオフにします。 | 着信の転送を停止して。 <br> 着信の転送をオフにして。
TurnForwardingOn | 着信の転送をオンにして。 | 着信を 3333 に転送 <br>  3333 への着信の転送をオンにして。
GetForwardingsStatus | 着信の転送の現在の状態を取得します。 | 着信の転送は有効? <br>   自分の通話状態がオンかオフかを教えて。
GetNotifications | 通知を受け取ります。 | 通知を開いて。 <br>   スマホに Facebook から通知が来ていないか確認して。
Goback | 前の手順に戻ります。 | Twitter に戻って <br>   1 ステップ戻って <br>   Go back
Ignore | 電話の着信を無視します。 | 応答しないで <br>   着信を無視して
IgnoreWithMessage | 電話の着信を無視して、代わりにテキストで返信します。 | 電話に出ないで、代わりにメッセージを送信して。 <br>   無視して、テキストを返信して。
Dial | 電話を発信します。 | Jim に電話 <br>   311 に電話をかけて。
FindSpeedDial | 電話番号が設定されている短縮番号、またはその逆を検索します。 | ダイヤル番号 5 は何? <br>   短縮番号は設定されている? <br>   941-5555-333 のダイヤル番号は何?
転送 | メールを転送します。 | このメールを Greg に転送して。
ReadAloud | メッセージまたは電子メールをユーザーに読み上げます。 | テキストを読んで。 <br>   メッセージの内容を読んで。
PressKey | キーパッドのボタンまたは番号を押します。 | アスタリスクを押して。 <br>   1 2 3 を押して。
QueryLastText | 最後のテキストまたはメッセージについて問い合わせます。 | 私に SMS を送ったのは誰? <br>   私に最近メッセージを送ったのは誰?
Redial | ある番号にリダイヤルまたは再発信します。 | リダイヤルして。 <br>   最後の発信をリダイヤルして。
Reject | 電話の着信を拒否します。 | 電話を拒否して <br>   今は応答できない <br>   ちょっと電話に出られない。後でかけ直す。
Reply | メッセージに返信します。 | Lore Hound に返信して。 <br>   今向かっていると入力して返信して。
SearchMessages | いくつかの条件でメッセージを検索します。 | Jane が送ってきたメールを検索して。
SendEmail | 電子メールを送信します。 この意図は、テキスト メッセージではなく電子メールに適用されます。 | Mike Waters に電子メール: Mike、先週のディナーは素晴らしかったですね。 <br>   Bob にメールを送信して。
SendMessage | テキスト メッセージまたはインスタント メッセージを送信します。 | Chris と Carol にテキストを送信して <br>   Facebook メッセージを Greg に送って <br>   
SetSpeedDial | 連絡先の電話番号の短縮ダイヤルを設定します。 | Carol に短縮ダイヤル 1 を設定して。 <br>   お母さんの短縮ダイヤルを設定して。
ShowCurrentNotification | 最近の通知を表示します。 | 新しい通知はある? <br> 通知を表示して。
ShowNext | テキスト メッセージや電子メールのリストなどの次の項目を表示します。 | 次の項目を表示して。 <br>   次のページに進んで。
ShowPrevious | テキスト メッセージや電子メールのリストなどの前の項目を表示します。 | 前の項目を表示して。 <br>   前へ。 <br>   前に戻って。
TurnSpeakerOff | スピーカー フォンをオフにします。 | スピーカー フォンを無効にして。 <br>   スピーカー フォンをオフにして。
TurnSpeakerOn | スピーカー フォンをオンにします。 | スピーカー フォン モード。 <br>   スピーカー フォンをオンにして。

### <a name="entities"></a>**エンティティ**
エンティティ名 | エンティティの種類 | 説明 | 例 | スロット
------|-------|----------|--------------|---------------
Attachment | simple | ユーザーがテキストまたはメールで送信する添付ファイル。 | OneNote から**ファイル**をメールで送信して。 <br> Katie にハウスキーピング **ドキュメント**を送信して。 | file <br> ドキュメント
AudioDeviceType | simple | オーディオ デバイスの種類 (スピーカー、ヘッドセット、マイクなど)。 | **ハンズフリー**で応答して。 <br> **スピーカーホン**でリダイヤルして。 | スピーカー <br> ハンズフリー <br> Bluetooth
Category | simple | メッセージまたはメールのカテゴリ。メール システムでは、カテゴリに明瞭な定義を与える必要があります。たとえば、"未読"、"フラグ" です。 明瞭な定義のない説明 ("新規" や "最近" など) はカテゴリではありません。 | メールを全部、**既読**にして。  <br> Paul 宛ての、新しい**優先度が高い**メール。 | 重要 <br> 優先度が高い <br> read
ContactAttribute | simple | ユーザーが問い合わせた連絡先の属性。| 来月、大事な**誕生日**はある? | 誕生日 <br> address <br> 電話番号
ContactName | personName  | 連絡先またはメッセージ受信者の名前。 | **Stevens** にメールを送って。 | Stevens
日付/時刻 | datetime | メールを受信した日時。 | **今日**のメールを読んで。 <br> **今日**、私にメールしたのは誰? <br> **午後 7 時**に電話したのは誰? | 本日 <br> 明日
DestinationPhone | simple | 通話または SMS 送信の相手。 | **家**に電話をかけて。 <br> **家**に SMS を送って。 | 家 <br> home
EmailAddress | email | 送信または問い合わせるメール アドレス。 | Megan.Flynn@MKF.com にメールを送って。<br> abc@outlook.com 
EmailSubject | simple、pattern.Any | 電子メールの件名として使用されるテキスト。 | David に送るメールを作る。件名は "**やあ**" にして。  | RE: 興味深い話
Key | simple | ユーザーが押すキー。 | **Space** キーを押して。 <br> **9** を押して。 | シャープ記号 <br> 星 <br> 8
Line | simple | メールまたは SMS を送信するときに使用す回線。 | 最後の **hotmail** を読んで。 <br> **携帯電話**で Peter を呼び出して。 <br> **職場**の電話でお父さんを呼び出して。| hotmail <br> Skype <br> British cell
SenderName | personName | 送信者の名前。 | **David** からのメールを読んで。 <br> Chanda からのメール。 | David <br> Chanda
FromRelationshipName | simple | 送信者のリレーションシップ名。 | **お父さん**からのメッセージを読んで。 <br> **お母さん**からの SMS を全部読んで。 | お父さん <br> お母さん 
Message | simple、pattern.Any |  電子メールまたはテキストとして送信するメッセージ。  | "**私は忙しいです**" というメールを送って。 | 私は忙しい
OrderReference | simple | 取得する項目を識別する、リスト内の順序または相対位置。 | 私が**最後**に送ったメッセージは何? <br> Nokia に届いた**最新の**メールを読んで。 <br> **新しい** SMS を読んで。 | last <br> latest <br> 最近 <br> 最新の
PositionReference | simple、ordinal | 取得する項目を識別する、リスト内の順序または相対位置。| 私が**最初**に送ったメッセージは何? <br> **3** つ目。| First (先頭へ) <br> 3 つ目
phoneNumber | phoneNumber | 通話または SMS 送信の電話番号。 | **4156845284** に SMS を送って。 | 3525214446
RelationshipName | simple | 連絡先またはメッセージ受信者のリレーションシップ名。 | **妻**にメールを送って。 | 妻
SearchTexts | simple、pattern.any | メールまたはメッセージをフィルター処理するために使用するテキスト | "**Surface Pro**" が含まれるメールをすべて検索して | Surface Pro
SpeedDial | simple | 短縮ダイヤル。 | **3 4 5** に電話して。 <br> 短縮ダイヤル **1** を設定。 | 345 <br> 5

## <a name="email"></a>**電子メール**
Email は *Communication* ドメインのサブドメインです。 主に、電子メール経由でメッセージを送受信する要求が含まれています。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
AddMore | 段階的な電子メールまたはテキストの作成の一部として、電子メールまたはテキストに追加します。 | メールの本文にさらに追加して。
Cancel | 操作を取り消します。 <br> ***注意**:より汎用的な Cancel の発話については、**Utilities** ドメインを参照してください。* | キャンセルして。 送らないで。 <br> 終了して。
CheckMessages | 条件を問わず、新しいメッセージ/メールがないか確認します。 条件がある場合、発話は *SearchMessage* 意図に属します。 | 受信トレイを確認して。 <br> 新しいメールは来ていない? 
Confirm | メールの処理と関連している進行中のアクションを確認します。 <br> ***注意**:より汎用的な Confirm の発話については、**Utilities** ドメインを参照してください。* | はい、送信して。 <br> このメールの送信を確定します。
削除 | いくつかの条件でフィルター処理したメールを削除します。 | メールをごみ箱に入れて。 <br> 受信トレイを空にして。 <br> Mary のメールを削除して。
ReadAloud | メッセージまたは電子メールをユーザーに読み上げます。 <br> ***注意**:より汎用的な ReadAloud の発話については、**Utilities** ドメインを参照してください。*  | Mary から届いたメールを読んで。
Reply | 現在のメールにメッセージを返信します。 | メールの返信を作成。 <br> "ありがとうございました。よろしくお願いします。John" と書いて返信して。
SearchMessages | 送信者の名前、時刻、件名など、いくつかの条件でメッセージを検索します。 | Nisheen から届いたメッセージを見せて。 <br> "ABC" というタイトルのメールを探して。
SendEmail | 電子メールを送信します。 | Mike にメール。Mike、先週のディナーは素晴らしかったですね。 <br> Bob にメールを送信して。
ShowNext | テキスト メッセージやメールのリストで次の項目を表示します。 <br> ***注意**:より汎用的な ShowNext の発話については、**Utilities** ドメインを参照してください。* | 次の項目を表示して。 <br> 次のページに進んで。
ShowPrevious | テキスト メッセージやメールのリストで前の項目を表示します。 <br> ***注意**:より汎用的な ShowPrevious の発話については、**Utilities** ドメインを参照してください。* | 前の項目を表示して。 <br> 前へ。 <br> 前に戻って。
転送 | メールを転送します。 | このメールを Greg に転送して。
AddFlag | メールにフラグを追加します。 | このメールにフラグを付けて。 <br> このメールにフラグを追加して。
QueryLastText | 最後のメールを問い合わせます。 | 私にメールを送ったのは誰? <br> 私に最近、メールしたのは誰?


### <a name="entities"></a>**エンティティ**
エンティティ名 | エンティティの種類 | 説明 | 例 | スロット
------|-------|----------|--------------|---------------
Attachment | simple | ユーザーがテキストまたはメールで送信する添付ファイル。 | OneNote から**ファイル**をメールで送信して。 <br> Katie にハウスキーピング **ドキュメント**を送信して。 | file <br> ドキュメント
ContactName | personName  | 連絡先またはメッセージ受信者の名前。 | **Stevens** にメールを送って。 | Stevens
Date | datetime | メールを受信した日付。 | **今日**のメールを読んで。 <br> **今日**、私にメールしたのは誰? | 本日
EmailAddress | email | 送信または問い合わせるメール アドレス。 | Megan.Flynn@MKF.com にメールを送って。<br> abc@outlook.com 
EmailSubject | simple、pattern.Any | 電子メールの件名として使用されるテキスト。 | David に送るメールを作る。件名は**やあ**にして。  | RE: 興味深い話
SenderName | personName | 送信者の名前。 | **David** からのメールを読んで。 <br> Chanda からのメール。 | David <br> Chanda
FromRelationshipName | simple | 送信者のリレーションシップ名。 | **お父さん**からのメッセージを読んで。 <br> **お母さん**からの SMS を全部読んで。 | お父さん <br> お母さん 
Message | simple、pattern.Any |  電子メールまたはテキストとして送信するメッセージ。  | "**私は忙しいです**" というメールを送って。 | 私は忙しい
Category | simple | メッセージまたはメールのカテゴリ。メール システムでは、カテゴリに明瞭な定義を与える必要があります。たとえば、"未読"、"フラグ" です。 明瞭な定義のない説明 ("新規" や "最近" など) はカテゴリではありません。 | メールを全部、**既読**にして。  <br> Paul 宛ての、新しい**優先度が高い**メール。 | 重要 <br> 優先度が高い <br> read
OrderReference | simple | 取得する項目を識別する、リスト内の順序または相対位置。 | 私が**最後**に送ったメッセージは何? <br> Nokia に届いた**最新の**メールを読んで。 <br> **新しい** SMS を読んで。 | last <br> latest <br> 最近 <br> 最新の
PositionReference | simple、ordinal | 取得する項目を識別する、リスト内の順序または相対位置。| 私が**最初**に送ったメッセージは何? <br> **3** つ目。| First (先頭へ) <br> 3 つ目
RelationshipName | simple | 連絡先またはメッセージ受信者のリレーションシップ名。 | **妻**にメールを送って。 | 妻
Time | datetime | Time | **今夜**、メールを送って。 | 今夜
SearchTexts | simple、pattern.any | メールまたはメッセージをフィルター処理するために使用するテキスト | "**Surface Pro**" が含まれるメールをすべて検索して | Surface Pro 
Line | simple | メールを送信するときに使用するライン。 | 最後の **hotmail** を読んで。 <br> **職場アカウント**からメールを送って。| hotmail <br> 職場アカウント 

## <a name="homeautomation"></a>**HomeAutomation**
HomeAutomation ドメインには、スマート ホーム デバイスの制御に関する意図とエンティティが用意されています。 主に、照明とエアコンに関連する制御コマンドをサポートしています。 他の電子機器でも使える機能があります。
### <a name="supported-devices-and-properties"></a>**サポートされているデバイスとプロパティ**
Device | properties
-------|---------
温度センサー | 気温
ライト、ランプ | オン/オフ、明るさ、色
テレビ、ラジオ | オン/オフ、音量
エアコン、サーモスタット | オン/オフ、温度、強さ
換気扇 | オン/オフ、強さ
コンセント、DVD プレーヤー、製氷機など | オン/オフ

### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
 TurnOff | ユーザーはデバイスまたは設定をオフにします。  | 照明を消して。 <br> <何か> を消して。 <br> <何か> オフ。
 TurnOn | デバイスをオンにするか、デバイスをオンにして特定の設定またはモードに設定します。 | 照明をつけて 50% にして。 <br> コーヒー メーカーの電源を入れて。 <br> コーヒー メーカーの電源を入れて。
 SetDevice | ユーザーはデバイスを特定の設定または種類に設定します。  | エアコンを 26 度に設定して。 <br> ライトを青にして。 <br> 常夜灯としてこのライトを呼び出して。
 QueryState | ユーザーはデバイスまたは設定の状態を問います。  | サーモスタットは何に設定されている? <br> エアコンはついてる? <br> 寝室の温度は?
 TurnUp | デバイスの設定または明るさを上げて。 | ライトを 75% 明るくして。 <br> ここの明るさを 10% 上げて。  <br> 居間をもっと暖かくして。
 TurnDown | ほの暗くしたり、温度やライトの明るさを下げたりしますが、デバイスをオフにすることはありません。 | 図書室を 44% 下げて。 <br> ほの暗くして。 <br> 部屋をもっと涼しくして。
### <a name="entities"></a>**エンティティ**
エンティティ名 | エンティティの種類 | 説明 | 例
-------|----------|--------------|---------------
DeviceName | List | デバイスのユーザー定義テキスト。 | 青<br> クリスマス <br> 机
DeviceType | List | サポートされているデバイス。 | 照明 <br> エアコン <br> 常夜灯
Location | simple | デバイスがある場所または部屋。 | 台所<br> 階下 <br> 寝室
NumericalChange | simple | 設定の増減量。 <br> <br> _スロットは turn_up 意図と turn_down 意図でのみ表示されます。_ | 3<br> 50%<br>
OrderReference | ordinal | このスロットの目的は、項目の選択をキャプチャすることです。 リスト上の項目の位置を示します。 | First (先頭へ)<br> 秒
Quantifier | List | Quantifier は特定のエンティティ インスタンスが参照される数を示します。 "全部" や "すべて" などです。 | All<br> すべて<br> すべて
Setting | シンプル | シーン、レベル、強度、色、モード、温度、デバイスの状態など、ユーザーがデバイスに指定する設定。 | 青<br> 72 <br> 暖房 
SettingType | List | ユーザーが関心を持っているデバイス設定。 | 気温<br> 
Time/Date |  datetime | デバイスを操作する時間と期間。| 5 分 <br> 午後 3 時
単位 | List | "度"、"パーセント"、"華氏"、"摂氏" などの単語にタグを付けるには、単位用語別にタグを付けてください。 | 度<br> Percent


## <a name="notes"></a>**メモ**
Note ドメインは、ユーザーがメモを作成したり、項目を書き留めたりする行為を支援します。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
AddToNote | 開いているメモに情報を追加します。 | 計画メモに追加し、プロジェクトについて上司に連絡して。
Clear | 既存のメモからすべての項目をクリアします。 | 休暇メモからすべての項目を削除して。 <br>読書メモを全部消去して。
Confirm | メモに関するアクションを確認します。 <br> ***注意**:この意図では主に、"Confirm" アクションは Note シナリオ向けとして含まれます。 "Confirm" でさらに一般的な表現が必要であれば、**Utilities** ドメインで "Confirm" 意図を活用してください。* | このメモでもかまいません。 <br>リスト上のすべての項目を保存することを確認します。
Create | 新しいメモを作成します。 | メモを作成して。 <br>Jason が 5 月第 1 週に上京することを忘れないようにメモして。 
削除 | メモ全体を削除します。 | 休暇メモを削除して。 <br>買い物メモを削除して。
ReadAloud | メモを読み上げて。 | 最初のメモを読んで。 <br>詳細を読んで。
閉じます | 現在のメモを閉じて。 | メモを閉じて。 <br>現在のメモを閉じて。
オープン | 既存のメモを開きます。 | 仕事メモを開いて。 <br>"計画" メモを開いて。
RemoveSentence | メモの文を削除します。 | 最後の文を削除して。 <br>買い物メモからポテトチップスを削除して。 <br>学校の購入メモからペンを削除して。
ChangeTitle | メモのタイトルを変更します。 | このメモの名前を "計画" にして。

### <a name="entities"></a>**エンティティ**
エンティティ名 | エンティティの種類 | 説明 | 例 
------- | ------- | ------- | -------
Text | simple、pattern.Any | メモまたはリマインダーのテキスト。 | 散歩前にストレッチ <br> 明日は長距離走
タイトル | simple、pattern.Any | メモのタイトル。 | 買い物メモ <br> 電話する相手 <br> To do
CreationDate | datetimeV2 | このスロットは、特定の日付/時刻ウィンドウ内で作成されたメモをユーザーが要求するときに使用されます。 | 
Quantifier | List | ユーザーが "全部"、"すべて"、または "あらゆる" 項目またはノート内のすべてのテキストに対して操作を実行するように求めるとき。 | すべて <br> 任意 <br> すべて
OrderReference | ordinal | "最初の"、"最後の"、"次の" などの項目でアクションを実行することをユーザーが要求します。 | first <br> last


## <a name="places"></a>**Places**
Places には、会社、施設、レストラン、公共の場所、住所などが含まれます。 このドメインでは、場所の検索や、公共の場所の情報 (所在地、営業時間、距離など) についての質問がサポートされています。 
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
MakeCall | 場所に電話をかけます。 | 1000/200 Rojas St のアップルビーズ。電話して。
FindPlace | 場所 (会社、施設、レストラン、公共の場所、住所) を検索します。 ただし、次の検索はできません。 <li> 企業名のみ:スターバックス <li> 場所の名前のみ:シアトル/ノルウェー <li> 料理、食品、または製品のみ:ピザ/グワカモーレ/イタリア料理 | 最寄りの図書館はどこですか? <br> シアトルのスターバックス。 <br> 最寄りの図書館はどこですか? <br> 
GetAddress | 場所の住所を確認します。 | Robson 通りの Guu の住所を表示して。 <br> 最寄りのスターバックスの住所を教えて
GetDistance | 現在の場所から特定の場所までの距離についてたずねます。 | ホリデイ インまでの距離はどのくらい?<br> ここからベルビュー スクエアまでの距離は? <br> タホまでの距離は?
GetPhoneNumber | 場所の電話番号を確認します。 | 最寄りのスターバックスの電話番号を教えて<br> ホーム デポの番号を教えて。 <br> ディズニーランドの電話番号。
GetPriceRange | ある場所で 1 人あたりの消費額の範囲をたずねます。 | シアトルにある J.Sushi の平均価格。 <br> シネプレックスは水曜日に半額になる? <br> シズラーのロブスター 1 尾付きのディナーはいくら?
GetStarRating | 場所の星評価を確認します。 | ズッカの評価は?<br> フレンチ ランドリーの星はいくつ?<br> モンテレイの水族館は良い?
GetHours | 場所の営業時間を確認します。 | セーフウェイの閉店時間はいつ?<br> ホーム デポの営業時間はいつ?<br> スターバックスはまだ開いてる?
GetReviews | 場所のレビューを確認します。 | チーズケーキ ファクトリのレビューを見せて。 <br> シネプレックスのレビューを読んで。 <br> Humperdinks に最近のレビューはある?
GetMenu | レストランのメニュー項目を確認します。 | ズッカはビーガン料理を出している? <br> シズラーにはどんなメニューがある? <br> アップルビーズのメニューを見せて。
RatePlace | 場所を評価します。 | キンバリーにある Maxi のピザに星 4 つを付けて。 <br> Tripadvisor で Bonefish に星 4 つを付けて。 <br> La Hacienda に星 4 つのレビューを作成して。
AddFavoritePlace | ユーザーが自分のお気に入りまたは MVP リストに、ある場所を追加したいと考えています。 | この場所をお気に入りに保存して。 <br> Best Buy をお気に入りに追加して。

### <a name="entities"></a>**エンティティ**
LUIS エンティティ | エンティティの種類 | 説明 | 例 | 発話の例
--------------|-------------|-------------|----------|-------------------
AbsoluteLocation | simple | 場所の位置情報または住所。 | パロ アルト <br> 300 112th Ave SE <br> シアトル | **1020 Middlefield Rd.** **パロ アルト**にある <br> **シアトル**にある粒餌の店 <br> ここから **300 112th Ave SE** までの距離を教えて。
Amenities | simple | 公共の場所の客観的な特性と利点。 | ウォーターフロント <br> 駐車無料 | カークランドにある**ウォーターフロント**という名前のシーフード レストラン。 <br> 近くに**無料の駐車場**はある?
Cuisine | simple | 食べ物、料理、または料理の国の種類。 | 中国語 <br> イタリア語 <br> 寿司 <br> 麺 <br> | **中華料理店**を探して。 <br> **寿司屋**の営業時間は? <br> 一番近くの**ステーキ** ハウスはどこ?
Date | datetime | ターゲット場所の日付の日時または期間。 | 明日 <br> 今日 <br> 午後 6 時 | **明日**、水族館は何時に閉まるの? <br> **午後 6 時**以降も開いている自転車店で一番近いところ。
Distance | ディメンション | ユーザーの現在位置から公共の場所までの距離。 | 24 km <br> 10 マイル | **15 マイル**以内の衣料品店 <br> **10 マイル**で行ける子供向けレストラン
MealType | List | 朝食や昼食などの食事の種類。 | 朝食 <br> 夕食 | **朝食**を検索して。シアトルのグリーンウッドで。 <br> **昼食**をとれる場所を見つけて。
Nearby | List | 場所または住所がはっきりしなくても近くの場所を説明します。 | 一番近いところ <br> この地域で <br> この付近で | **一番近く**のインド料理店を探して。 <br> **地元**のウェザースプーンはどこ? <br> **この近く**に良いレストランはある?
OpenStatus | List | 場所が開いているか閉じているかを示します。 | オープン <br> closed | 今日、ヨーグルト ランドが**閉店**するのは何時? <br> コストコの**営業**時間は?
PlaceName | simple | 会社、レストラン、観光地、または施設である目的地の名前。 場所名には、一般的に使用される場合、場所の種類が含まれることがあります。 | セントラル パーク <br> セーフウェイ <br> ウォルマート| **セーフウェイ**の薬局は何時に開店? <br> **ウォルマート**は開いてる?
PlaceType | simple | 地域の会社、レストラン、観光地、または施設である目的地の種類。 | restaurant <br> opera <br> cinema | ケンブリッジの**映画館** <br> 近くに**レストラン**はある?
PriceRange | simple | その場所の製品またはサービスの価格範囲。 | 安い <br> 費用効果の高い <br> 高い | **料金が手頃な**家電修理店を見つけて。 <br> 現在開いている**安い**ピザ屋さんはどこ?
Product | simple | 場所が提供している製品。 | 衣類 <br> テレビ | **食品**を買う一番良い店はどこ? <br> イースト キルブライドで**テレビ**を売っている店を探して。
Rating | simple | 場所の評価。 | 5 つ星 <br> top <br> 良い | 明日外食するけど、どこか**良い**店はある? <br> アムステルダムで**最高**のレストラン <br> 人気のピザ屋さんを**上位** 3 つリストアップして。


## <a name="restaurantreservation"></a>**RestaurantReservation**
Restaurant Reservation ドメインでは、レストランの予約処理の意図がサポートされています。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
Reserve | レストランの予約を要求します。 | ズッカで予約して。2 人用のテーブル。今夜。 <br> レストランを予約して。明日。 <br> パロ アルトで予約を取って。3 人用のテーブル。7 時。 <br> レッド ロブスターに予約を取って。3 人用のテーブル。
DeleteReservation | レストランの予約を取り消します (削除します)。 | 明日のズッカの予約を取り消して。 <br> レッド ロブスター、午後 7 時の予約を削除して。 今度の金曜日の。 <br> ズッカの予約が不要になったからキャンセルして。
ChangeReservation | レストランの既存予約の時刻、場所、人数を変更します。 | 予約を午後 9 時に変更して。 <br> 予約をズッカからレッド ロブスターに変更して。 <br> ズッカの予約を 6 名ではなく 7 名に。
Confirm | 予約関連のアクションを確認します。 <br> ***注意**:この意図では主に、"Confirm" アクションは Restaurant Reservation シナリオ向けとして含まれます。 "Confirm" でさらに一般的な表現が必要であれば、**Utilities** ドメインで "Confirm" 意図を活用してください。* | はい。パスタ メーカー、今夜午後 8 時で予約しました。 <br> はい、予約してください。 <br> スシ バーの予約を確認。
FindReservationDetail | 既存の予約の詳細情報を表示します。 | サンディエゴ、ルネサンスの予約を検索して。 <br> 明日の予約を見せて。 <br> ズッカの予約の詳細を見せて。
FindReservationWhere | 予約の位置情報を確認します。 | 予約しているズッカの場所は? <br> 明日の予約の場所を見せて。
FindReservationWhen | 予約の正確な時刻を確認します。 | 明日のズッカの予約は何時? <br> レッド ロブスターの予約は何時?
Reject | ユーザーは、予約管理に関する仮想アシスタントの提案を拒否します。 <br> ***注意**: より汎用的な Reject の発話については、**Utilities** ドメインを参照してください。* | イベントを設定する必要はありません。 | いいえ、予約を変更しないで。 <br> いいえ、予約しないで。間違えました。

### <a name="entities"></a>**エンティティ**
LUIS エンティティ | エンティティの種類 | 説明 | 例
-------|------|---------|-------------------
Address | simple | 予約のイベントの場所または住所。 | パロ アルト<br>300 112th Ave SE<br>シアトル
Atmosphere | List | レストランの雰囲気の説明。 | ロマンティック<br>カジュアル<br>グループ向け<br>素敵
Cuisine | simple | 食べ物、料理、または料理の国の種類。 | 中国語<br>イタリア語<br>メキシコ<br>寿司<br>麺<br>ステーキ
MealType | List | 予約に関連する食事の種類。 | 朝食<br>夕食<br>昼食<br>夜食
PlaceName | simple | レストランの名前 | ズッカ<br>チーズケーキ ファクトリ<br>レッド ロブスター
Rating | simple | 場所またはレストランの評価。 | 5 つ星<br>3 つ星<br>4 つ星
NumberPeople | simple | 予約の人数 | 3<br>6
Time | datetime| レストランの予約時刻 | 明日<br>今夜<br>午後 7 時<br>クリスマス イブ


## <a name="todo"></a>**ToDo**
_ToDo_ ドメインには、ユーザーが ToDo 項目を追加、マーク、削除できる各種タスク リストが用意されています。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
AddToDo | ユーザーは任意のリストタイプのタスク項目を追加することを望みます。 |  牛乳を買うことを通知して。 <br> "牛乳を買う" という項目を To-Do リストに追加して。
Confirm | ユーザーは実行中のアクションを確認することを望みます。 この発話には、ある操作を確定する "はい" など、明示的なシグナルが含まれます。 <br> <br> ***注意**:この意図では主に、"Confirm" アクションは ToDo シナリオ向けとして含まれます。 "Confirm" でさらに一般的な表現が必要であれば、**Utilities** ドメインで "Confirm" 意図を活用してください。* | タスクを削除して。 <br> このタスクを追加します。 <br> はい、それを希望します。
Cancel | ユーザーは実行中のアクションを取り消すことを望みます。  <br> <br> ***注意**:この意図では主に、"Confirm" アクションは Restaurant Reservation シナリオ向けとして含まれます。 "Confirm" でさらに一般的な表現が必要であれば、**Utilities** ドメインで "Confirm" 意図を活用してください。* | いいえ、削除して。 <br> タスクをキャンセルして。 <br> いいえ、追加しないで。
DeleteToDo | その指図を参照してタスク内の項目を削除するか、To-Do 項目を全部削除します。 | すべてのタスクを削除して。 <br> 2 つ目を削除して。
MarkToDo | その指図またはタスクの内容を参照し、タスクを完了としてマークします。 | "牛乳を買う" というタスクを完了にして。 <br> タスクの読み取りを完了。 <br> 全部完了。
ShowNextPage | リストは複数のページに分割され、表示されます。 ユーザーに次のページのリスト項目を見せます。 | 買い物リストの次のページを見せて。 <br> 次は何? <br> 次の手順
ShowPreviousPage | ユーザーに前のページのリスト項目を見せます。 | 前を表示。 <br> 前のタスクを確認する必要があります。
ShowToDo | To-Do リストの全項目を表示します。 | 買い物リストを見せて。 <br> 食料品リストを見せて。

### <a name="entities"></a>**エンティティ**
LUIS エンティティ | エンティティの種類 | 説明 | 例
-------|------|---------|-------------------
ContainsAll | simple | タスク リスト内のすべての (あらゆる) 項目を示します。 | タスクを**全部**削除して。 <br> **全部**完了にして。
ordinal | ordinal | 項目を順番や数字で参照します。 | **3 つ目**を完了にして。 <br> **1 つ目**のタスクを削除して。
ListType | simple | タスク リストの種類。  | **買い物**リストに靴を追加して。
FoodOfGrocery | List | 食品項目のリストを検出します。 | **牛乳**を買うことを通知して。 <br> 食料品の買い物リストに**牛肉**を追加して。
TaskContent | simple、pattern.any | タスクのコンテンツを検出します。 | **お母さんに電話する**と通知して。 <br> **John の誕生日祝い**を To-Do リストに追加して。


## <a name="utilities"></a>**Utilities**
_Utilities_ ドメインは、すべての _LUIS_ 事前構築済みモデル間の一般的なドメインで、さまざまなシナリオに共通の意図と発話が含まれています。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
 Cancel | アクション/要求/タスク/サービスをキャンセルします。 | キャンセルして。 <br> 気にしないで、忘れて。
 Confirm | アクションを確認します。 | 承知しました <br> はい、お願いします <br> Confirm.
 Reject | ユーザーは仮想アシスタントの提案を拒否します。 | いいえ
 FinishTask | ユーザーが開始したタスクを完了します。 | 終わりました。 <br> これで全部です。 <br> 完了。
 GoBack | 1 ステップ前に戻るか、前のステップに戻ります。 | 1 ステップ戻って。 <br> 戻って。
 [Help] | ヘルプを要求します。 | 助けて。 <br> ヘルプを開いて。
 Repeat | アクションを繰り返します。 | もう一度言って。
 ShowNext | 次の項目を見せます (伝えます)。 | 次の項目を表示して。
 ShowPrevious | 系列内の前の項目を表示します。 | 前の項目を表示して。
 StartOver | アプリを再起動するか、新しいセッションを開始します。 | 再起動して。
 Stop | 仮想アシスタントに話すことを止めるように伝えます。  | この読み上げを停止してください。
 SelectAny | 画面に表示されている項目または結果をどれでも良いので選択するようにユーザーが求めます。 | どれか 1 つ。 <br> どれか 1 つ選択して。
 SelectNone | ユーザーは既存の項目から選択せず、他の選択肢を求めます。 | 他の提案をお願い。 <br> 全部なし。
  SelectItem | 画面に表示されている項目または結果を選択するようにユーザーが求めます。 | 3 つ目を選択して。 <br> 右上の項目を選択して。
 エスカレート | 問題に対処するようにカスタマー サービスに要請します。 | 担当者と話せる?
 ReadAloud | ユーザーのために <何か> を読み上げます。 | このページを読んで。 <br> 読み上げて。

### <a name="entities"></a>**エンティティ**
LUIS エンティティ | エンティティの種類 | 説明 | 例
------------|-------------|-------------|---------
ordinal | ordinal | 項目を順番や数字で参照します。 | **2** つ目。 <br> **次**の項目。
number | number | ユーザーが必要とする項目の数量 | 次の **3** つの項目
DirectionalReference | simple | 画面上のどこに項目があるかを示す参照ポイント。 | 右の項目 <br> upper

## <a name="weather"></a>**Weather**
Weather ドメインでは、場所や時間を指定して気象条件や注意報を調べたり、気象条件別に時間を調べたりすることに重点を置いています。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
 CheckWeatherValue | 天気予報または過去の情報に基づき、ある土地の天候または天候関連の要素について問います。 <br> 天候関連の要素: <li> 温度 </li> <li> 雨/雪/降水 </li> <li> 乾燥/湿気/湿度 </li> <li> 霧 </li> <li> 紫外線指数 </li> <li> 雨量/降雪量 </li> | 北京の大気質指数は? <br> 3 月のシアトルの降水予想は? <br> ハワイの過去最高気温。
 CheckWeatherTime | ある土地の天気予報または過去の天気に関連する要素の時期をたずねます。 | 雨はいつ降るの? <br> カナダに旅行するおすすめの時期 <br> ミネソタの一番暑い月は?
 QueryWeather | "はい、いいえ、たぶん" という応答が予想される、特定の土地の天気や天候関連の要素についてたずねます。あるいは、活動に対し、天候に基づく助言を求めます。 | 明日は雨降るの? <br> 今日は晴れる? <br> アラスカに行くとしたら、5 月は早すぎる?
 ChangeTemperatureUnit | 摂氏、ケルビン、華氏など、天候単位を変更します。 | 摂氏なら何度? <br> ケルビンなら何度?
 GetWeatherAdvisory | 特定の土地に対する天気関連の助言や警告を取得します。 | メンフィスの天気のアドバイスはある? <br> この地域に天候に関する注意報はある?

### <a name="entities"></a>**エンティティ**
LUIS エンティティ | エンティティの種類 | 説明 | 例
------------|-------------|-------------|---------
Location | 地理 | 天候についてたずねる絶対的または暗黙的な場所。 | パロ アルト<br>上海<br>シアトル<br>デルビナ<br>
日付/時刻   | datetime | 天候についてたずねる日時または期間。 | 11 月<br>1 時間に 1 回<br>朝<br>今週末<br>10 日<br>
AdditionalWeatherCondition | list | 風速や風向など、天候に関する補足説明。 | direction<br>速い<br>速度
Historic | simple | 過去の平均や近似など、過去の天候を説明します。 | 過去<br>歴史上/歴史<br>季節<br>最適な時期<br>観測史上
PrecipitationUnit | ディメンション | 降雪量または降雨量。 | 5 インチ<br>6 センチ
SuitableFor | simple | ユーザーが天候に基づく行動助言を求めるときに一般的となる、ある天候下での人間の行動を説明します。 | 上着<br>傘<br>水泳
TemperatureUnit |温度 | 温度 | 摂氏 18 度<br>ケルビン 7 度
WeatherRange | simple | ある期間を対象とする、温度、風、その他の天候の特定の条件。 | maximum<br>高<br>低<br>平均高<br>最高
WeatherCondition | simple | 天候条件の説明 | 晴れ<br>雨<br>雨が降っています<br>温度<br>雪<br>hot (暖かい)
WindDirectionUnit | list | 風向を示す言葉 | north<br>南<br>東<br>西<br>北東


## <a name="web"></a>**Web**
Web ドメインには、Web サイトを検索するための意図とエンティティが用意されています。
### <a name="intents"></a>**意図**
意図の名前 | 説明 | 例
---------|----------|---------
 WebSearch | 指定 Web サイトへの移動要求または検索エンジンでの検索要求。 | google.com で surface を検索して。 <br> ネットで誕生日の歌を見つけて <br> www.twitter.com にアクセスして。

### <a name="entities"></a>**エンティティ**
LUIS エンティティ | エンティティの種類 | 説明 | 例
------------|-------------|-------------|---------
SearchEngine | List | ユーザーが使用を望む検索エンジン。 | Bing <br> Google
SearchText | simple、pattern.Any | ユーザーが検索を希望するテキスト。 <br> _"in" の後の Web サイトが検索エンジンではない場合、"friends in facebook" に SearchText のタグを付けます。URL にも SearchText のタグを付けます。_ | 映画 <br> ディープ ラーニング <br> トム クルーズ
Link | url | Web サイト リンク。 | www.twitter.com

