---
title: マネージド インスタンスの既存の仮想ネットワークを構成する
description: この記事では、Azure SQL Database Managed Instance をデプロイできる既存の仮想ネットワークとサブネットの構成方法について説明します。
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab
ms.date: 03/17/2020
ms.openlocfilehash: 50b832baa9253f47b5f10980ae1764c9425ed4d7
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "79476951"
---
# <a name="configure-an-existing-virtual-network-for-azure-sql-database-managed-instance"></a>Azure SQL Database Managed Instance の既存の仮想ネットワークを構成する

Azure SQL Database Managed Instance は、Azure [仮想ネットワーク](../virtual-network/virtual-networks-overview.md)と Managed Instance 専用のサブネット内にデプロイする必要があります。 [Managed Instance の仮想ネットワーク要件](sql-database-managed-instance-connectivity-architecture.md#network-requirements)に従って構成されていれば、既存の仮想ネットワークとサブネットを使用できます。

次のいずれかのケースに該当する場合は、この記事で説明されているスクリプトを使用してネットワークを検証および変更できます。

- まだ構成されていない新しいサブネットがある。
- サブネットが[要件](sql-database-managed-instance-connectivity-architecture.md#network-requirements)に準拠しているかどうかがわからない。
- 変更を加えた後も、サブネットが[ネットワーク要件](sql-database-managed-instance-connectivity-architecture.md#network-requirements)に準拠していることを確認したい。

> [!Note]
> Azure Resource Manager デプロイ モデルを使用して作成された仮想ネットワーク内にのみマネージド インスタンスを作成できます。 クラシック デプロイ モデルを使用して作成された Azure 仮想ネットワークはサポートされていません。 [マネージド インスタンスのサブネットのサイズの決定](sql-database-managed-instance-determine-size-vnet-subnet.md)に関する記事のガイドラインに従って、サブネットのサイズを計算します。 サブネットは、いったんリソースを配置するとサイズを変更できなくなります。
>
> マネージド インスタンスを作成した後は、マネージド インスタンスまたは VNet を別のリソース グループまたはサブスクリプションに移動することはできません。

## <a name="validate-and-modify-an-existing-virtual-network"></a>既存の仮想ネットワークを検証および変更する

既存のサブネット内にマネージド インスタンスを作成する場合は、次の PowerShell スクリプトでサブネットを準備することをお勧めします。

```powershell
$scriptUrlBase = 'https://raw.githubusercontent.com/Microsoft/sql-server-samples/master/samples/manage/azure-sql-db-managed-instance/delegate-subnet'

$parameters = @{
    subscriptionId = '<subscriptionId>'
    resourceGroupName = '<resourceGroupName>'
    virtualNetworkName = '<virtualNetworkName>'
    subnetName = '<subnetName>'
    }

Invoke-Command -ScriptBlock ([Scriptblock]::Create((iwr ($scriptUrlBase+'/delegateSubnet.ps1?t='+ [DateTime]::Now.Ticks)).Content)) -ArgumentList $parameters
```

このスクリプトでは、3 つの手順でサブネットを準備します。

1. 検証:選択された仮想ネットワークとサブネットが Managed Instance のネットワーク要件と照らして検証されます。
2. 確認:Managed Instance のデプロイに備えてサブネットを準備するために必要な一連の変更がユーザーに示されます。 また、同意も求められます。
3. 準備:仮想ネットワークとサブネットが適切に構成されます。

## <a name="next-steps"></a>次のステップ

- 概要については、[マネージド インスタンス](sql-database-managed-instance.md)に関するページを参照してください。
- 仮想ネットワークを作成し、マネージド インスタンスを作成して、データベース バックアップからデータベースを復元する方法を示すチュートリアルについては、[Azure SQL Database マネージド インスタンスの作成](sql-database-managed-instance-get-started.md)に関するページを参照してください。
- DNS の問題については、[カスタム DNS の構成](sql-database-managed-instance-custom-dns.md)に関するページを参照してください。
