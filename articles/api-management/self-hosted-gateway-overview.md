---
title: セルフホステッド Azure API Management ゲートウェイの概要 | Microsoft Docs
description: 組織がハイブリッド環境やマルチクラウド環境で API を管理するのに、セルフホステッド Azure API Management ゲートウェイがどのように役立つかを説明します。
services: api-management
documentationcenter: ''
author: vlvinogr
manager: gwallace
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 10/31/2019
ms.author: apimpm
ms.openlocfilehash: 415f0e209e607a863d715b1a66a2435603a662f0
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "73510556"
---
# <a name="self-hosted-api-management-gateway-overview"></a>セルフホステッド Azure API Management ゲートウェイの概要

この記事では、セルフホステッド ゲートウェイ機能によってハイブリッドおよびマルチクラウドの API 管理を実現する方法について説明します。また、アーキテクチャの概要を紹介し、基本的な機能を取り上げます。

> [!NOTE]
> セルフホステッド ゲートウェイ機能はプレビュー段階です。 プレビュー期間中、セルフホステッド ゲートウェイは、Developer レベルと Premium レベルでのみ追加料金なしで利用できます。 Developer レベルは、1 つのセルフホステッド ゲートウェイのデプロイに制限されます。

## <a name="hybrid-and-multi-cloud-api-management"></a>ハイブリッドおよびマルチクラウドでの API 管理

セルフホステッド ゲートウェイ機能は、ハイブリッド環境とマルチクラウド環境に対する API Management サポートを拡張し、オンプレミスや複数のクラウドにわたってホストされている API を、組織が Azure の単一 API Management サービスから効率的かつ安全に管理できるようにします。

セルフホステッド ゲートウェイを使用すると、お客様は、コンテナー化されたバージョンの API Management ゲートウェイ コンポーネントを、API をホストしているのと同じ環境に柔軟にデプロイできるようになります。 すべてのセルフホステッド ゲートウェイは、フェデレーションされている API Management サービスから管理されます。これによりお客様には、内部と外部のすべての API にわたる可視性と統合された管理エクスペリエンスが提供されます。 ゲートウェイを API の近くに配置することで、お客様は API トラフィック フローを最適化し、セキュリティとコンプライアンスの要件に対応することができます。

それぞれの API Management サービスは、以下の主なコンポーネントで構成されています。

-   API として公開される管理プレーン。Azure portal、PowerShell、およびその他のサポートされているメカニズムによってサービスを構成するために使用されます。
-   ゲートウェイ (またはデータ プレーン) は、API 要求のプロキシ処理、ポリシーの適用、テレメトリの収集を行います
-   API を利用するための検索、学習、およびオンボードのために開発者によって使用される開発者ポータル

既定では、これらのすべてのコンポーネントが Azure にデプロイされ、API を実装しているバックエンドがホストされている場所に関係なく、すべての API トラフィック (下図では実線の黒い矢印として示されています) が Azure 経由で流れるようになります。 このモデルの運用上のシンプルさは、待機時間の増加やコンプライアンスの問題という代償によって実現されており、場合によって追加のデータ転送料金が発生します。

![セルフホステッド ゲートウェイを使用しない場合の API トラフィック フロー](media/self-hosted-gateway-overview/without-gateways.png)

セルフホステッド ゲートウェイをバックエンド API 実装と同じ環境に配置し、それらを API Management サービスに追加することで、API トラフィックをバックエンド API に直接流すことができます。それにより、待機時間が短縮され、データ転送コストが最適化されて、実装がホストされている場所に関係なく、組織内のすべての API を 1 か所で管理および検出するというメリットを維持しながらコンプライアンスを実現できます。

![セルフホステッド ゲートウェイを使用する場合の API トラフィック フロー](media/self-hosted-gateway-overview/with-gateways.png)

## <a name="packaging-and-features"></a>パッケージと機能

セルフホステッド ゲートウェイは、すべての API Management サービスの一部として Azure にデプロイされるマネージド ゲートウェイと機能的に同等の、コンテナー化されたバージョンです。 セルフホステッド ゲートウェイは、Microsoft Container Registry から Linux ベースの Docker コンテナーとして入手できます。 これは、デスクトップ、サーバー クラスター、またはクラウド インフラストラクチャで実行されている Docker、Kubernetes、またはその他任意のコンテナー オーケストレーション ソリューションに配置できます。

> [!IMPORTANT]
> マネージド ゲートウェイで使用できる機能の一部は、プレビューではまだ利用できません。 特に重要なものは、イベント ハブ ポリシーへのログ記録、Service Fabric の統合、ダウンストリーム HTTP/2 です。 セルフホステッド ゲートウェイで組み込みのキャッシュを使用できるようにする計画はありません。

## <a name="connectivity-to-azure"></a>Azure への接続性

セルフホステッド ゲートウェイには、ポート 443 での Azure への送信 TCP/IP 接続が必要です。 各セルフホステッド ゲートウェイは、1 つの API Management サービスに関連付けられている必要があり、管理プレーンを介して構成されます。 セルフホステッド ゲートウェイは、以下の場合に Azure への接続を使用します。

-   1 分ごとのハートビート メッセージ送信による状態の報告
-   構成の更新の、定期的チェック (10 秒ごと) と、入手可能な場合は常に実行する適用
-   要求ログとメトリックの Azure Monitor への送信 (これを行うよう構成されている場合)
-   Application Insights へのイベントの送信 (これを行うよう設定されている場合)

Azure への接続性が失われると、セルフホステッド ゲートウェイは構成の更新の受信、状態の報告、テレメトリのアップロードができなくなります。

セルフホステッド ゲートウェイは、"静的に失敗する" ように設計されており、Azure への接続が一時的に失われても問題は生じません。 これは、ローカル構成のバックアップが有効になっていてもいなくても展開できます。 前者の場合、セルフホステッド ゲートウェイは定期的に、コンテナーまたはポッドに接続された永続的ボリュームに構成のバックアップ コピーを保存します。

構成のバックアップがオフになっていて、Azure への接続が断たれると、以下のようになります。

-   実行中のセルフホステッド ゲートウェイは、構成のメモリ内コピーを使用して機能し続けます
-   停止されていたセルフホステッド ゲートウェイは起動できなくなります

構成のバックアップがオンになっていて、Azure への接続が断たれると、以下のようになります。

-   実行中のセルフホステッド ゲートウェイは、構成のメモリ内コピーを使用して機能し続けます
-   停止されていたセルフホステッド ゲートウェイは、構成のバックアップ コピーの使用を開始します

接続が復元されると、停止の影響を受けた各セルフホステッド ゲートウェイは、関連付けられている API Management サービスに自動的に再接続し、ゲートウェイが "オフライン" になっていた間に発生したすべての構成の更新をダウンロードします。

## <a name="next-steps"></a>次のステップ

-   [このトピックの追加の背景情報についてのホワイトペーパーを読む](https://aka.ms/hybrid-and-multi-cloud-api-management)
-   [Docker にセルフホステッド ゲートウェイをデプロイする](api-management-howto-deploy-self-hosted-gateway-to-docker.md)
-   [Kubernetes にセルフホステッド ゲートウェイをデプロイする](api-management-howto-deploy-self-hosted-gateway-to-k8s.md)
