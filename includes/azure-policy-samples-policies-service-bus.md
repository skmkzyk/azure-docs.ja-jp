---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 03/12/2020
ms.author: dacoulte
ms.openlocfilehash: 10cd98a350ceb622f07fcb6c85477260b2c7cb60
ms.sourcegitcommit: 0947111b263015136bca0e6ec5a8c570b3f700ff
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/24/2020
ms.locfileid: "79382167"
---
|Name |説明 |効果 |Version |GitHub |
|---|---|---|---|---|
|[Service Bus の名前空間から RootManageSharedAccessKey 以外のすべての承認規則を削除する必要がある](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fa1817ec0-a368-432a-8057-8371e17ac6ee) |Service Bus クライアントでは、名前空間内のすべてのキューおよびトピックへのアクセスを提供する名前空間レベルのアクセス ポリシーを使用してはいけません。 最小限の特権セキュリティ モデルに合わせるために、キューとトピックについてはエンティティ レベルでアクセス ポリシーを作成し、特定のエンティティのみへのアクセスを提供する必要があります |Audit、Deny、Disabled |1.0.1 |[リンク](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditNamespaceAccessRules_Audit.json)
|[Service Bus で診断ログを有効にする必要がある](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff8d36e2f-389b-4ee4-898d-21aeb69a0f45) |診断ログが有効になっていることを監査します。 これにより、セキュリティ インシデントが発生した場合やお使いのネットワークが侵害された場合に、調査目的で使用するアクティビティ証跡を再作成できます |AuditIfNotExists、Disabled |2.0.0 |[リンク](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Service%20Bus/ServiceBus_AuditDiagnosticLog_Audit.json)
