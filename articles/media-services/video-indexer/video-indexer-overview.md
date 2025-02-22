---
title: Azure Media Services Video Indexer とは
titleSuffix: Azure Media Services
description: このトピックでは、Azure Media Services Video Indexer サービスの概要を説明します。
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 09/06/2019
ms.author: juliako
ms.openlocfilehash: e3f60b5fb0693e40c9db040f7b14f487fce8f68e
ms.sourcegitcommit: 65131f6188a02efe1704d92f0fd473b21c760d08
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 09/10/2019
ms.locfileid: "70860244"
---
# <a name="what-is-video-indexer"></a>Video Indexer とは

Azure Media Services Video Indexer は、Azure Media Analytics、Azure Search、Cognitive Services (Face API、Microsoft Translator、Computer Vision API、Custom Speech Service など) を基盤として構築されたクラウド アプリケーションです。 以下で説明する Video Indexer のビデオとオーディオのモデルを使用して、ビデオから分析情報を抽出することができます。
  
## <a name="video-insights"></a>ビデオの分析情報

- **顔検出**:ビデオに表示される顔を検出し、グループ化します。
- **著名人の識別**:Video Indexer では、世界中のリーダー、男優、女優、アスリート、研究者、ビジネス リーダーおよび技術リーダーなど、100 万人を超える著名人を自動的に識別します。 これらの著名人に関するデータは、IMDB や Wikipedia などのさまざまな有名な web サイトにも記載されています。
- **アカウントベースの顔識別**:Video Indexer では、特定のアカウントのモデルをトレーニングします。 その後、トレーニングされたモデルに基づいてビデオ内の顔を認識します。 詳細については、「[Video Indexer Web サイトから人物モデルをカスタマイズする](customize-person-model-with-website.md)」と「[Video Indexer API を使用して人物モデルをカスタマイズする](customize-person-model-with-api.md)」をご覧ください。
- **顔のサムネイルの抽出** ("最適な顔"):顔の各グループでキャプチャされた最適な顔を (品質、サイズ、および正面位置に基づいて) 自動的に識別し、それをイメージ アセットとして抽出します。
- **ビジュアル テキストの認識** (OCR):ビデオ内に視覚的に表示されるテキストを抽出します。
- **ビジュアル コンテンツ モデレーション**:成人向けやわいせつなビジュアルを検出します。
- **ラベルの識別**:表示されるビジュアル オブジェクトとアクションを識別します。
- **シーンのセグメント化**: 視覚的な手掛かりに基づいて、ビデオ内でシーンが変化するタイミングを判定します。シーンは単一のイベントを表し、意味的に関連する一連の連続したショットで構成されます。 
- **ショット検出**: 視覚的な手掛かりに基づいて、ビデオ内のショットが変化するタイミングを判定します。ショットは、同じ動画カメラから撮影された一連のフレームです。 詳細については、「[Scenes, shots, and keyframes](scenes-shots-keyframes.md)」(シーン、ショット、キーフレーム) を参照してください。
- **黒フレームの検出**:ビデオに表示された黒フレームを識別します。
- **キーフレームの抽出**:ビデオ内の安定したキーフレームを検出します。
- **ローリング クレジット**: テレビ番組や映画の終わりにあるローリング クレジットの始まりと終わりを識別します。
- **アニメーション キャラクターの検出** (プレビュー): [Cognitive Services の Custom Vision](https://azure.microsoft.com/services/cognitive-services/custom-vision-service/) との統合によって、アニメ化されたコンテンツのキャラクターの検出、グループ化、および認識を行います。 詳細については、「[アニメーション キャラクターの検出](animated-characters-recognition.md)」を参照してください。
- **編集ショット タイプの検出**: タイプに基づくショットのタグ付け (ワイド ショット、ミディアム ショット、クローズアップ、エクストリーム クローズアップ、2 ショット、複数の人物、屋外、室内など)。 詳細については、「[編集ショット タイプの検出](scenes-shots-keyframes.md#editorial-shot-type-detection)」を参照してください。

## <a name="audio-insights"></a>オーディオの分析情報

- **自動言語検出**:主な音声言語を自動的に識別します。 英語、スペイン語、フランス語、ドイツ語、イタリア語、中国語 (簡体字)、日本語、ロシア語、ポルトガル語 (ブラジル) などの言語がサポートされています。 言語を確実に識別できない場合、Video Indexer では音声言語が英語と想定されます。 詳細については、[言語識別モデル](language-identification-model.md)に関する記事を参照してください。
- **複数言語の音声識別と文字起こし** (プレビュー):音声から異なるセグメントにある音声言語を自動的に識別し、メディア ファイルの各セグメントを文字起こしするために送信し、文字起こしを結合して元の 1 つの統合された文字起こしを作成します。 詳細については、「[複数言語のコンテンツを自動的に識別および文字起こしする](multi-language-identification-transcription.md)」を参照してください。
- **音声の文字起こし**:12 の言語で音声をテキストに変換します。拡張機能を使用できます。 英語、スペイン語、フランス語、ドイツ語、イタリア語、中国語 (簡体字)、日本語、アラビア語、ロシア語、ポルトガル語 (ブラジル)、ヒンディー語、韓国語などの言語がサポートされています。
- **字幕**:VTT、TML、SRT という 3 つの形式で字幕を作成します。
- **2 チャネル処理**:自動検出、トランスクリプトの分離、1 つのタイムラインへの結合を行います。
- **ノイズリダクション**:(Skype フィルターに基づいて) テレフォニー音声やノイズの多い録音を明瞭にします。
- **トランスクリプトのカスタマイズ** (CRIS):音声テキスト変換のカスタム モデルをトレーニングして、業界固有のトランスクリプトを作成します。 詳細については、「[Video Indexer Web サイトから言語モデルをカスタマイズする](customize-language-model-with-website.md)」と「[Video Indexer API を使用して言語モデルをカスタマイズする](customize-language-model-with-api.md)」をご覧ください。
- **話者の列挙**:どの話者がどの言葉をいつ話したかをマップして認識します。
- **話者の統計情報**:話者の音声率の統計情報を提供します。
- **テキストのコンテンツ モデレーション**:音声トランスクリプト内の明示的なテキストを検出します。
- **音声効果**:拍手、発言、沈黙などの音声効果を識別します。
- **感情の検出**:音声 (話されている内容) と口調 (話し方) に基づいて感情を特定します。  この感情は、喜び、悲しみ、怒り、または恐怖の可能性があります。
- **翻訳**:音声トランスクリプトの、54 の異なる言語への翻訳を作成します。

## <a name="audio-and-video-insights-multi-channels"></a>オーディオとビデオの分析情報 (マルチ チャンネル)

1 つのチャンネルでインデックスを付けるときは、これらのモデルの部分的な結果を利用できます

- **キーワードの抽出**:音声と視覚テキストからキーワードを抽出します。
- **名前付きエンティティの抽出**:自然言語処理 (NLP) を使用して、音声および視覚テキストからブランド、場所、および人物を抽出します。
- **トピックの推論**:トランスクリプトから主なトピックを推論します。 第 1 レベルの IPTC 分類が含まれています。
- **成果物**:各モデルについて、"次のレベルの詳細情報" 成果物の豊富なセットを抽出します。
- **センチメント分析**:音声と視覚テキストから、ポジティブ、ネガティブ、ニュートラルのセンチメントを識別します。
 
Video Indexer での処理と分析が終わったら、ビデオの分析情報をレビュー、キュレーション、検索、および発行できます。

役割がコンテンツ マネージャーまたは開発者であっても、Video Indexer サービスがあればニーズに対応できます。 コンテンツ マネージャーは Video Indexer Web ポータルを使用して、1 行のコードも記述することなく、サービスを使用できます。[Video Indexer Web サイトの使用開始](video-indexer-get-started.md)についてのページを参照してください。 開発者は API を活用して、コンテンツを大規模に処理できます。[Video Indexer REST API の使用](video-indexer-use-apis.md)に関する記事をご覧ください。 このサービスでは、顧客はウィジェットを使用してビデオ ストリームと、独自のアプリケーションで抽出された分析情報を発行することもできます。「[Embed visual widgets in your application](video-indexer-embed-widgets.md)」(アプリケーションでビジュアル ウィジェットを埋め込む) を参照してください。

既存の AAD、LinkedIn、Facebook、Google、または MSA アカウントを使用して、サービスにサインアップすることができます。 詳細については、[概要](video-indexer-get-started.md)に関するページを参照してください。

## <a name="scenarios"></a>シナリオ

Video Indexer が役に立ついくつかのシナリオを以下に示します。

- 検索 – ビデオから抽出された分析情報を使用して、ビデオ ライブラリ全体での検索エクスペリエンスを向上させることができます。 たとえば、話されている語句と顔にインデックスを作成すると、特定の人物が特定の単語をいつ話したかや、2 人の人物がいつ会っていたかを検索できるようになります。 ビデオからのこのような分析情報に基づいた検索は、通信社、教育機関、放送局、エンターテイメント コンテンツの所有者、エンタープライズ LOB アプリにとって利用価値があり、一般には、ユーザーが検索の対象にするビデオ ライブラリを保有するすべての業界が対象になります。
- コンテンツ作成 – ビデオから分析情報を抽出し、組織アーカイブの既存のコンテンツから、予告編、ソーシャル メディア コンテンツ、ニュース クリップなどのコンテンツを効果的に作成するのに役立ちます。 
- 収益化 – Video Indexer は、ビデオの収益価値の向上に役立ちます。 たとえば、広告収入に依存している業界 (ニュース メディア、ソーシャル メディアなど) では、抽出した分析情報を広告サーバーへの追加のシグナルとして利用することで、より関連性の高い広告を提供できます (スポーツ シューズの広告を、水泳競技ではなくフットボールの試合の途中に表示すれば、関連性が高まります)。
- ユーザー エンゲージメント – Video Indexer で解明した分析情報は、ユーザーに関連のあるビデオ モーメントをユーザーに提供してユーザー エンゲージメントを向上させるために使用できます。 たとえば、最初の 30 分間は球について説明し、次の 30 分間はピラミッドについて説明する教育ビデオについて考えてみましょう。 ピラミッドについて読んでいる学生にとっては、このビデオの 30 分マーカーから始まるようにビデオが配置されれば役立ちます。

## <a name="next-steps"></a>次の手順

これで、Video Indexer の使用を開始する準備ができました。 詳細については、次の記事を参照してください。

- [Video Indexer Web サイトの使用を開始する](video-indexer-get-started.md)
- [Video Indexer REST API を使用してコンテンツを処理する](video-indexer-use-apis.md)
- [アプリケーションにビジュアル ウィジェットを埋め込む](video-indexer-embed-widgets.md)
