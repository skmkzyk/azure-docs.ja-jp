---
title: トランザクション レプリケーション
description: Azure SQL Database で単一データベース、プールされたデータベース、インスタンス データベースを使用した SQL Server のトランザクション レプリケーションの使用について学習します。
services: sql-database
ms.service: sql-database
ms.subservice: data-movement
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: carlrab
ms.date: 02/08/2019
ms.openlocfilehash: 41dd336bdb74fbe745ab48ebd3c168af0492ae2c
ms.sourcegitcommit: 2ec4b3d0bad7dc0071400c2a2264399e4fe34897
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 03/27/2020
ms.locfileid: "75691017"
---
# <a name="transactional-replication-with-single-pooled-and-instance-databases-in-azure-sql-database"></a>Azure SQL Database での単一データベース、プールされたデータベース、インスタンス データベースを使用したトランザクション レプリケーション

トランザクション レプリケーションは、Azure SQL Database または SQL Server のテーブルからリモート データベースに配置されているテーブルにデータをレプリケートするための、Azure SQL Database、SQL Server の機能です。 この機能を使用すると、さまざまなデータベース内の複数のテーブルを同期させることができます。

## <a name="when-to-use-transactional-replication"></a>トランザクション レプリケーションを使用するタイミング

トランザクション レプリケーションは次のシナリオで役立ちます。
- データベース内の 1 つまたは複数のテーブルで行われた変更を発行して、これらの変更のためにサブスクライブした 1 つまたは複数の SQL Server または Azure SQL データベースに配布します。
- いくつかの分散データベースは同時状態を保持します。
- 変更を継続的に発行することによって、データベースを 1 つの SQL Server またはマネージド インスタンスから別のデータベースに移行します。

## <a name="overview"></a>概要

トランザクション レプリケーションの主要なコンポーネントは、次の図のとおりです。  

![SQL Database を使用したレプリケーション](media/replication-to-sql-database/replication-to-sql-database.png)

**パブリッシャー**は、ディストリビューターに更新プログラムを送信することで一部のテーブル (アーティクル) で加えられた変更を発行するインスタンスまたはサーバーです。 オンプレミスの SQL サーバーから Azure SQL データベースへの発行は、次のバージョンの SQL Server でサポートされます。

- SQL Server 2019 (プレビュー)
- SQL Server 2016 から SQL 2017
- SQL Server 2014 SP1 CU3 以上 (12.00.4427)
- SQL Server 2014 RTM CU10 (12.00.2556)
- SQL Server 2012 SP3 以上 (11.0.6020)
- SQL Server 2012 SP2 CU8 (11.0.5634.0)
- Azure でのオブジェクトへの発行をサポートしていないその他のバージョンの SQL Server では、[データの再発行](https://docs.microsoft.com/sql/relational-databases/replication/republish-data)方法を利用して新しいバージョンの SQL Server にデータを移動することができます。 

**ディストリビューター**は、パブリッシャーからアーティクル内の変更を収集してサブスクライバーに配布するインスタンスまたはサーバーです。 ディストリビューターにできるのは、Azure SQL Database マネージド インスタンスまたは SQL Server (パブリッシャーのバージョン以上の任意のバージョン) です。 

**サブスクライバー**は、パブリッシャーで加えられた変更を受信しているインスタンスまたはサーバーです。 Azure SQL Database または SQL Server データベースでは、サブスクライバーは単一データベース、プールされたデータベース、インスタンス データベースのいずれかになります。 単一データベースまたはプールされたデータベース上のサブスクライバーは、プッシュ サブスクライバーとして構成する必要があります。 

| Role | 単一データベースとプールされたデータベース | インスタンス データベース |
| :----| :------------- | :--------------- |
| **発行元** | いいえ | はい | 
| **ディストリビューター** | いいえ | はい|
| **プル サブスクライバー** | いいえ | はい|
| **プッシュ サブスクライバー**| はい | はい|
| &nbsp; | &nbsp; | &nbsp; |

  >[!NOTE]
  > ディストリビューターがインスタンス データベースとサブスクライバーでない場合、プル サブスクリプションはサポートされていません。 

さまざまな[レプリケーションの種類](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication)があります。


| レプリケーション | 単一データベースとプールされたデータベース | インスタンス データベース|
| :----| :------------- | :--------------- |
| [**標準トランザクション**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/transactional-replication) | はい (サブスクライバーとしてのみ) | はい | 
| [**スナップショット**](https://docs.microsoft.com/sql/relational-databases/replication/snapshot-replication) | はい (サブスクライバーとしてのみ) | はい|
| [**マージ レプリケーション**](https://docs.microsoft.com/sql/relational-databases/replication/merge/merge-replication) | いいえ | いいえ|
| [**ピア ツー ピア**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/peer-to-peer-transactional-replication) | いいえ | いいえ|
| [**双方向**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/bidirectional-transactional-replication) | いいえ | はい|
| [**更新可能なサブスクリプション**](https://docs.microsoft.com/sql/relational-databases/replication/transactional/updatable-subscriptions-for-transactional-replication) | いいえ | いいえ|
| &nbsp; | &nbsp; | &nbsp; |

  >[!NOTE]
  > - 以前のバージョンを使用してレプリケーションを構成しようとすると、エラー番号 MSSQL_REPL20084 (プロセスはサブスクライバーに接続できませんでした) および MSSQ_REPL40532 (ログインによって要求されたサーバー \<name> を開くことができません。 ログインに失敗しました) のエラーが発生する可能性があります。
  > - Azure SQL Database のすべての機能を使用するには、最新バージョンの [SQL Server Management Studio (SSMS)](https://docs.microsoft.com/sql/ssms/download-sql-server-management-studio-ssms) および [SQL Server Data Tools (SSDT)](https://docs.microsoft.com/sql/ssdt/download-sql-server-data-tools-ssdt) を使用する必要があります。
  
  ### <a name="supportability-matrix-for-instance-databases-and-on-premises-systems"></a>インスタンス データベースとオンプレミス システムのサポート性マトリックス
  インスタンス データベースのレプリケーション サポータビリティ マトリックスは、SQL Server オンプレミスのものと同じです。 
  
| **発行元**   | **ディストリビューター** | **サブスクライバー** |
| :------------   | :-------------- | :------------- |
| SQL Server 2019 | SQL Server 2019 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/>  |
| SQL Server 2017 | SQL Server 2019 <br/>SQL Server 2017 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 |
| SQL Server 2016 | SQL Server 2019 <br/>SQL Server 2017 <br/> SQL Server 2016 | SQL Server 2019 <br/> SQL Server 2017 <br/>SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 |
| SQL Server 2014 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>| SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |
| SQL Server 2012 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> | SQL Server 2016 <br/> SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 | 
| SQL Server 2008 R2 <br/> SQL Server 2008 | SQL Server 2019 <br/> SQL Server 2017 <br/> SQL Server 2016 <br/> SQL Server 2014 <br/>SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 |  SQL Server 2014 <br/> SQL Server 2012 <br/> SQL Server 2008 R2 <br/> SQL Server 2008 <br/>  |
| &nbsp; | &nbsp; | &nbsp; |

## <a name="requirements"></a>必要条件

- 接続では、レプリケーション参加者間で SQL 認証を使用します。 
- レプリケーションで使用される作業ディレクトリの Azure ストレージ アカウント共有。 
- Azure ファイル共有にアクセスするために、マネージド インスタンス サブネットのセキュリティ規則でポート 445 (TCP アウトバウンド) を開く必要があります。 
- ポート 1433 (TCP アウトバウンド) は、パブリッシャーおよびディストリビューターがマネージド インスタンス上にあり、サブスクライバーがオンプレミスにある場合は、開かれている必要があります。
- すべての種類のレプリケーション参加者 (パブリッシャー、ディストリビューター、プル サブスクライバー、プッシュ サブスクライバー) をマネージド インスタンスに配置できますが、パブリッシャーとディストリビューターは両者ともクラウドに配置するか、または両者ともオンプレミスに配置する必要があります。
- パブリッシャー、ディストリビューター、および/またはサブスクライバーが異なる仮想ネットワークに存在する場合、パブリッシャーとディストリビューターの間にVPNピアリングがあり、かつ/またはディストリビューターとサブスクライバーの間にVPNピアリングがあるように、各エンティティ間にVPNピアリングを確立する必要があります。 


>[!NOTE]
> - ディストリビューターがインスタンス データベースでサブスクライバーがオンプレミスであるときに発信ネットワーク セキュリティ グループ (NSG) ポート 445 がブロックされている場合、Azure Storage ファイルに接続するときは、エラー 53 が発生する可能性があります。 [vNet NSG を更新して](/azure/storage/files/storage-troubleshoot-windows-file-connection-problems)、この問題を解決します。 


### <a name="compare-data-sync-with-transactional-replication"></a>データ同期とトランザクション レプリケーションの比較

| | データ同期 | トランザクション レプリケーション |
|---|---|---|
| 長所 | - アクティブ/アクティブのサポート<br/>- オンプレミスと Azure SQL Database 間で双方向 | - 待ち時間の短縮<br/>- トランザクションの整合性<br/>- 移行後に既存のトポロジの再利用 |
| 短所 | - 5 分以上の待機時間<br/>- トランザクションの整合性なし<br/>- パフォーマンスへの影響が大きい | - Azure SQL Database の単一データベースまたはプールされたデータベースから発行できない<br/>- 高いメンテナンス コスト |
| | | |

## <a name="common-configurations"></a>一般的な構成

一般に、パブリッシャーとディストリビューターは、クラウドかオンプレミスのいずれかに存在する必要があります。 次の構成がサポートされています。 

### <a name="publisher-with-local-distributor-on-a-managed-instance"></a>マネージド インスタンス上のパブリッシャーとローカル ディストリビューター

![パブリッシャーおよびディストリビューターとしての 1 つのインスタンス](media/replication-with-sql-database-managed-instance/01-single-instance-asdbmi-pubdist.png)

パブリッシャーとディストリビューターは、1 つのマネージド インスタンス内に構成され、その他のマネージド インスタンス、単一データベース、プールされたデータベース、またはオンプレミスの SQL Server に変更が配布されます。 

### <a name="publisher-with-remote-distributor-on-a-managed-instance"></a>マネージド インスタンス上のパブリッシャーとリモート ディストリビューター

この構成では、1 つのマネージド インスタンスがもう 1 つのマネージド インスタンス上に配置されたディストリビューターに変更を発行します。このディストリビューターが多数のソース マネージド インスタンスにサービスを提供し、マネージド インスタンス、単一データベース、プールされたデータベース、または SQL Server 上の 1 つまたは複数のターゲットに変更を配布します。

![パブリッシャーおよびディストリビューターの独立したインスタンス](media/replication-with-sql-database-managed-instance/02-separate-instances-asdbmi-pubdist.png)

2 つのマネージド インスタンス上にパブリッシャーとディストリビューターが構成されています。 この構成には、次のようないくつかの制約があります。 

- 2 つのマネージド インスタンスが同じ vNet 上にあります。
- 2 つのマネージド インスタンスが同じ場所にあります。


### <a name="publisher-and-distributor-on-premises-with-a-subscriber-on-a-single-pooled-and-instance-database"></a>オンプレミスのパブリッシャーおよびディストリビューターと、単一データベース、プールされたデータベース、インスタンス データベース上のサブスクライバー 

![サブスクライバーとしての Azure SQL DB](media/replication-with-sql-database-managed-instance/03-azure-sql-db-subscriber.png)
 
この構成では、Azure SQL Database (単一データベース、プールされたデータベース、インスタンス データベース) がサブスクライバーです。 この構成では、オンプレミスから Azure への移行がサポートされます。 サブスクライバーが単一データベースまたはプールされたデータベース上にある場合は、プッシュ モードにする必要があります。  

## <a name="with-failover-groups"></a>フェイルオーバーグループを使う

Geo レプリケーションが、**フェールオーバーグループ** の**パブリッシャー** または [ディストリビューター](sql-database-auto-failover-group.md) 上で有効化されている場合、マネージドインスタンス管理者は、フェイルオーバー発生後、古いプライマリ上のすべてのアプリケーションをクリーンアップして、新しいプライマリ上でそれらを再構成する必要があります。 このシナリオでは、次のアクティビティが必要です。

1. データベース上で実行されているレプリケーション ジョブがある場合は、すべて停止します。
2. パブリッシャーからサブスクリプションのメタデータを削除するには、パブリッシャー データベース上で次のスクリプトを実行します。

   ```sql
   EXEC sp_dropsubscription @publication='<name of publication>', @article='all',@subscriber='<name of subscriber>'
   ```             
 
1. サブスクライバーからサブスクリプションのメタデータを削除します。 サブスクライバー インスタンス上のサブスクリプション データベースで、次のスクリプトを実行します：

   ```sql
   EXEC sp_subscription_cleanup
      @publisher = N'<full DNS of publisher, e.g. example.ac2d23028af5.database.windows.net>', 
      @publisher_db = N'<publisher database>', 
      @publication = N'<name of publication>'; 
   ```                

1. パブリッシャーからすべてのレプリケーション オブジェクトを強制的に削除するには、発行されたデータベースで次のスクリプトを実行します。

   ```sql
   EXEC sp_removedbreplication
   ```

1. 元のプライマリ インスタンスから以前のディストリビューターを強制的に削除します (ディストリビューターを持つために使用されていた以前のプライマリにフェールバックする場合)。 以前のディストリビューター マネージド インスタンスのマスター データベース上で次のスクリプトを実行します。

   ```sql
   EXEC sp_dropdistributor 1,1
   ```

Geo レプリケーションが、フェイルオーバーの **サブスクライバー** インスタンスで有効になっている場合、パブリケーションは、サブスクライバーマネージド インスタンスのフェールオーバーグループのリスナーエンドポイントに接続するように構成されている必要があります。 フェールオーバーが発生した場合、マネージド インスタンス管理者による以降のアクションは、発生したフェールオーバーの種類によって異なります： 

- データ損失のないフェールオーバーの場合、フェールオーバー後もレプリケーションは機能し続けます。 
- データ損失を伴うフェイルオーバーの場合、レプリケーションも機能します。 失われた変更を再度レプリケートします。 
- データ損失のあるフェールオーバーでは、データ損失がディストリビューションデータベースの保有期間外の場合、マネージド インスタンス管理者はサブスクリプションデータベースを再初期化する必要があります。 

## <a name="next-steps"></a>次のステップ

- [MI パブリッシャーとサブスクライバーの間でのレプリケーションを構成する](replication-with-sql-database-managed-instance.md)
- [MI パブリッシャーと MI ディストリビューター、および SQL Server サブスクライバーの間のレプリケーションを構成する](sql-database-managed-instance-configure-replication-tutorial.md)
- [パブリケーションを作成します](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)。
- Azure SQL Database サーバー名をサブスクライバーとして ([ など)、Azure SQL Database 名を宛先データベースとして (](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription)AdventureWorks`N'azuresqldbdns.database.windows.net` など) 使用して、**プッシュ サブスクリプションを作成します**。 )


トランザクションレプリケーションの構成の詳細については、次のチュートリアルを参照してください：



## <a name="see-also"></a>参照  

- [MI およびフェールオーバー グループを使用したレプリケーション](sql-database-managed-instance-transact-sql-information.md#replication)
- [SQL Database へのレプリケーション](replication-to-sql-database.md)
- [マネージド インスタンスへのレプリケーション](replication-with-sql-database-managed-instance.md)
- [パブリケーションを作成する](https://docs.microsoft.com/sql/relational-databases/replication/publish/create-a-publication)
- [プッシュ サブスクリプションを作成する](https://docs.microsoft.com/sql/relational-databases/replication/create-a-push-subscription/)
- [レプリケーションの種類](https://docs.microsoft.com/sql/relational-databases/replication/types-of-replication)
- [監視 (レプリケーション)](https://docs.microsoft.com/sql/relational-databases/replication/monitor/monitoring-replication)
- [サブスクリプションを初期化する](https://docs.microsoft.com/sql/relational-databases/replication/initialize-a-subscription)  
