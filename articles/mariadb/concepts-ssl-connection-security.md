---
title: SSL 接続 - Azure Database for MariaDB
description: SSL 接続を適切に使用できるように、Azure Database for MariaDB と関連アプリケーションを構成するための情報
author: ajlam
ms.author: andrela
ms.service: mariadb
ms.topic: conceptual
ms.date: 03/10/2020
ms.openlocfilehash: 36532575645d135a7abe7239798b6f2abc4246f2
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/28/2020
ms.locfileid: "79477070"
---
# <a name="ssl-connectivity-in-azure-database-for-mariadb"></a>Azure Database for MariaDB での SSL 接続
Azure Database for MariaDB では、Secure Sockets Layer (SSL) を使用して、データベース サーバーをクライアント アプリケーションに接続できます。 データベース サーバーとクライアント アプリケーション間に SSL 接続を適用すると、サーバーとアプリケーション間のデータ ストリームが暗号化されて、"man in the middle" 攻撃から保護されます。

## <a name="default-settings"></a>既定の設定
既定では、MariaDB サーバーへの接続時に SSL 接続が要求されるようにデータベース サービスを構成する必要があります。  可能な場合は、SSL オプションを無効にしないことをお勧めします。

Azure portal や CLI を使用して新しい Azure Database for MariaDB サーバーをプロビジョニングする場合、SSL 接続の適用が既定で有効になります。

さまざまなプログラミング言語の接続文字列が Azure Portal に表示されます。 これらの接続文字列には、データベースへの接続に必要な SSL パラメーターが含まれます。 Azure Portal で、お使いのサーバーを選択します。 **[設定]** の見出しにある **[接続文字列]** を選択します。 SSL パラメーターはコネクタによって異なります ("ssl=true"、"sslmode=require"、"sslmode=required" など)。

アプリケーションの開発時に SSL 接続を有効または無効にする方法については、[SSL の構成方法](howto-configure-ssl.md)に関するページをご覧ください。

## <a name="next-steps"></a>次のステップ
- [サーバー ファイアウォール規則](concepts-firewall-rules.md)について説明します。
- [SSL の構成](howto-configure-ssl.md)方法について説明します。
