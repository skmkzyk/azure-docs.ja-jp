---
title: トラブルシューティング - Personalizer
description: この記事では、Personalizer についてよく寄せられる質問とその回答を示します。
ms.topic: troubleshooting
ms.date: 02/26/2020
ms.author: diberry
ms.openlocfilehash: 904953f028eb31afe42cf477ac05be43e8b72a4d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "80336024"
---
# <a name="personalizer-troubleshooting"></a>Personalizer のトラブルシューティング

この記事では、Personalizer についてよく寄せられる質問とその回答を示します。

## <a name="transaction-errors"></a>トランザクションのエラー

<details>
<summary><b>サービスから HTTP 429 (要求が多すぎます) という応答が返されました。どうすればよいですか。</b></summary>

**回答**: Personalizer インスタンスの作成時に無料の価格レベルを選択した場合、許容される Rank 要求の数にクォータ制限が存在します。 Azure portal の Personalizer リソースの [メトリック] ペインで Rank API の API 呼び出しレートを確認し、選択した価格レベルのしきい値を超える呼び出しボリュームになりそうな場合は、[価格レベル] ペインで価格レベルを調整してください。

</details>

<details>
<summary><b>Rank API または Reward API で 5xx エラーが発生します。どうすればよいですか。</b></summary>

**回答**: これらの問題をユーザーが意識する必要はありません。 問題が継続的に発生する場合は、Azure portal で Personalizer リソースの **[サポート + トラブルシューティング]** セクションにある **[新しいサポート リクエスト]** を選択して、サポートにお問い合わせください。

</details>

## <a name="learning-loop"></a>学習ループ

<details>
<summary>
<b>Personalizer がないシステムでは、学習ループは 100% の一致を達成しません。これをどのように修正すればよいですか?</b></summary>

**回答**: 学習ループで目標が達成されない理由は次のとおりです。
* Rank API 呼び出しで送信された特徴量が十分ではない
* 送信された特徴にバグがある (タイムスタンプなどの集計されていない特徴データが Rank API に送信されたなど)
* ループ処理にバグがある (イベントの報酬データを Reward API に送信していないなど)

修正するには、ループに送信された特徴を変更して処理を変更するか、報酬が Rank の応答の品質についての正しい評価であることを確認する必要があります。

</details>

<details>
<summary>
<b>学習ループで学習が行われていないようです。これをどのように修正すればよいですか?</b></summary>

**回答**: 学習ループでは、ランキング呼び出しによる優先順位の設定を効果的に行う前に、数千の報酬呼び出しが必要です。

学習ループの現在の動作に不安がある場合は、[オフライン評価](concepts-offline-evaluation.md)を実行し、修正された学習ポリシーを適用してください。

</details>

<details>
<summary><b>何度やっても、すべての項目がまったく同じ確率のランク結果が返されます。Personalizer が学習していることは、どうすれば確認できますか?</b></summary>

**回答**: Personalizer は、開始直後でモデルが "_空_" であるときや、Personalizer ループをリセットした時点でモデルがまだ "**モデルの更新頻度**" 期間内にあるとき、Rank API の結果として同じ確率を返します。

新しい更新期間が始まって、更新されたモデルが使用されると、確率が変化します。

</details>

<details>
<summary><b>学習ループによる学習は進行していますが、何も学習していないように見えます。Rank の結果の品質もあまり良くありません。どうすればよいですか。</b></summary>

**回答**:
* その Personalizer リソース (学習ループ) について、Azure portal で 1 回の評価を完了し、適用したことを確認します。
* Reward API を介してすべての報酬が送信され、処理されたことを確認します。

</details>


<details>
<summary><b>学習ループが定期的に更新され、データのスコア付けに使用されていることは、どうすればわかりますか?</b></summary>

**回答**: モデルの最終更新時刻は、Azure portal の **[モデルと学習設定]** ページで確認できます。 古いタイムスタンプが表示される場合、Rank と Reward の呼び出しを送信していないことが原因と考えられます。 サービスに受信データがなければ、学習は更新されません。 学習ループの更新頻度が不十分であると感じる場合は、ループの **[モデルの更新頻度]** を編集できます。

</details>

## <a name="offline-evaluations"></a>オフライン評価

<details>
<summary><b>オフライン評価の特徴量の重要度から、数百から数千の項目を含んだ長大なリストが返されます。なぜでしょうか?</b></summary>

**回答**: 一般に、タイムスタンプ、ユーザー ID、または、その他なんらかの粒度の細かい特徴が送信されたことが原因で発生します。

</details>

<details>
<summary><b>オフライン評価を作成したところ、ほとんど間を置かずに正常完了しました。なぜですか?結果がまったく表示されません。</b></summary>

**回答**: オフライン評価には、その期間のイベントから得られるトレーニング済みのモデル データが使用されます。 評価の開始時刻から終了時刻までの期間内のデータを送信しなかった場合は、結果が得られることなく完了します。 Personalizer にイベントが送信済みであることがわかっている時間範囲を選択して、新しいオフライン評価を送信してください。

</details>


## <a name="learning-policy"></a>学習ポリシー

<details>
<summary><b>学習ポリシーをインポートするにはどうすればよいですか?</b></summary>

**回答**: [学習ポリシーの概念](concept-active-learning.md#understand-learning-policy-settings)と、新しい学習ポリシーを[適用する方法](how-to-manage-model.md)をご覧ください。 学習ポリシーを選択したくない場合は、[オフライン評価](how-to-offline-evaluation.md)を使用すれば、現在のイベントに基づいて、学習ポリシーを提示することができます。

</details>

## <a name="security"></a>Security

<details>
<summary><b>ループの API キーが漏えいしました。どうすればよいですか。</b></summary>

**回答**: もう一方のキーを使用するようにクライアントをスワップした後で、キーを再生成することができます。 2 つのキーを使用することで、ダウンタイムを生じさせることなく、徐々にキーを伝達することができます。 この作業は、セキュリティ対策として定期的に実施することをお勧めします。

</details>

## <a name="next-steps"></a>次のステップ

[モデルの更新頻度を構成する](how-to-settings.md#model-update-frequency)