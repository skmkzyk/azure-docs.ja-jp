---
title: Cognitive Services Text Analytics リソースを作成する
titleSuffix: Azure Cognitive Services
description: Cognitive Services Text Analytics リソースを作成する方法について説明します。
services: cognitive-services
author: IEvangelist
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 11/21/2019
ms.author: dapine
ms.openlocfilehash: d3b65174f3d161e0b8780a890f297d9d1c812b44
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "74383455"
---
## <a name="create-a-cognitive-services-text-analytics-resource"></a>Cognitive Services Text Analytics リソースを作成する

1. [Azure portal](https://portal.azure.com) にサインインする
1. **[Create a resource]\(リソースを作成)** を選択して、**AI + Machine Learning** > **Text Analytics** に移動します。
   または、[[Create Text Analytics]\(Text Analytics の作成\)](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) に移動します。
1. すべての必須設定を入力します。

    |設定|値|
    |--|--|
    |Name|名前 (2 から 64 文字) を入力します。|
    |サブスクリプション|適切なサブスクリプションを選択します。|
    |Location|近くの場所を選択します。|
    |Pricing tier| **S** (Standard 価格レベル) を入力します。|
    |Resource group|使用可能なリソース グループを選択します。|

1. **[作成]** を選択して、リソースが作成されるまで待ちます。 ブラウザーが、新たに作成されたリソースのページに自動的にリダイレクトされます。
1. 構成された `endpoint` と API キーを収集します。

    |ポータルでの [リソース] タブ|設定|値|
    |--|--|--|
    |**概要**|エンドポイント|エンドポイントをコピーします。 `https://northeurope.api.cognitive.microsoft.com/text/analytics/v2.0` のように表示されます。|
    |**[キー]**|API キー|2 つのキーのうちの 1 つをコピーします。 スペースやダッシュを含まない 32 文字の英数字文字列 <`xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`> です。|
