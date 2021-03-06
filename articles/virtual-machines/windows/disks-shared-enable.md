---
title: Azure マネージド ディスクに対して共有ディスクを有効にする
description: 複数の VM 間で共有できるように、共有ディスク (プレビュー) を使用して Azure マネージド ディスクを構成する
author: roygara
ms.service: virtual-machines-windows
ms.topic: conceptual
ms.date: 02/18/2020
ms.author: rogarana
ms.subservice: disks
ms.openlocfilehash: e7ada64a50d6ace6ea4d34db87e0501d0311071d
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "77471699"
---
# <a name="enable-shared-disk"></a>共有ディスクを有効にする

この記事では、Azure マネージド ディスクに対して共有ディスク (プレビュー) 機能を有効にする方法について説明します。 Azure 共有ディスク (プレビュー) は、マネージド ディスクを複数の仮想マシン (VM) に同時に接続できるようにする Azure マネージド ディスクの新機能です。 マネージド ディスクを複数の VM に接続すると、新規にデプロイするか、既存のクラスター化されたアプリケーションを Azure に移行することができます。 

共有ディスクが有効になっているマネージド ディスクの概念的な情報については、「[Azure 共有ディスク](disks-shared.md)」を参照してください。
[!INCLUDE [virtual-machines-enable-shared-disk](../../../includes/virtual-machines-enable-shared-disk.md)]