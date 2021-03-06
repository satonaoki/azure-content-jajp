<properties
   pageTitle="SQL Data Warehouse でデータベースをユーザー エラーから復旧する |Microsoft Azure"
   description="SQL Data Warehouse でデータベースをユーザー エラーから復旧するための手順です。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="sahaj08"
   manager="barbkess"
   editor=""/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="03/28/2016"
   ms.author="sahajs;barbkess;sonyama"/>

# SQL Data Warehouse でのデータベースのユーザー エラーからの復旧

SQL Data Warehouse には、意図しないデータの破損または削除の原因となるユーザー エラーから復旧するための主な機能が 2 つあります。

- ライブ データベースの復元
- 削除されたデータベースの復元

これらの機能はいずれも、同じサーバー上の新しいデータベースに復元を実行します。復元先サーバーに新しいデータベース用の十分な DTU 容量があることが重要です。このクォータの増加を要求する場合は、[サポートに連絡][]してください。


## ライブ データベースの復元
Azure SQL Data Warehouse サービスは、少なくとも 8 時間ごとにデータベースのスナップショットを取得し、それらを 7 日間保持して復元ポイントの個別セットを提供することで、すべてのライブ データベースを保護します。データベース スナップショットは、データベースを一時停止または削除したときにも作成され、7 日間保持されます。ユーザー エラーが意図しないデータ変更の原因になっている場合は、保有期間内の復元ポイントのいずれかにデータベースを復元できます。


### Azure ポータル

Azure ポータルを使用して復元するには、次の手順に従います。

1. [Azure ポータル][]にログインします。
2. 画面の左側にある **[参照]** をクリックし、**[SQL データベース]** を選択します。
3. 目的のデータベースに移動し、それを選択します。
4. データベース ブレードの上部にある **[復元]** をクリックします。
5. 新しい**データベース名**を指定し、**[復元ポイント]** を選択して、**[作成]** をクリックします。
6. データベースの復元処理が開始され、**[通知]** を使用して処理を監視することができます。


### PowerShell

Azure PowerShell を使用して、プログラムによってデータベースの復元を実行します。Azure PowerShell モジュールをダウンロードするには、[Microsoft Web Platform Installer](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409) を実行します。Get-Module -ListAvailable -Name AzureRM.Sql を実行することで、バージョンを確認できます。この記事は、Microsoft AzureRM.Sql PowerShell バージョン 1.0.5 に基づいています。

データベースを復元するには、[Restore-AzureRmSqlDatabase][] コマンドレットを使用します。

1. Windows PowerShell を開きます。
2. Azure アカウントに接続して、アカウントに関連付けられているすべてのサブスクリプションを一覧表示します。
3. 復元するデータベースを含むサブスクリプションを選択します。
4. データベースの復元ポイントを表示します。
5. RestorePointCreationDate を使用して、目的の復元ポイントを選択します。
6. 目的の復元ポイントに、データベースを復元します。
7. 復元されたデータベースがオンラインであることを確認します。

```Powershell

Login-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

# List the last 10 database restore points
((Get-AzureRMSqlDatabaseRestorePoints -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>").RestorePointCreationDate)[-10 .. -1]

# Or list all restore points
Get-AzureRmSqlDatabaseRestorePoints -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>" 

# Pick desired restore point using RestorePointCreationDate
$PointInTime = "<RestorePointCreationDate>"

# Get the specific database to restore
$Database = Get-AzureRmSqlDatabase -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

# Restore database from a restore point
$RestoredDatabase = Restore-AzureRmSqlDatabase –FromPointInTimeBackup –PointInTime $PointInTime -ResourceGroupName $Database.ResourceGroupName -ServerName $Database.ServerName -TargetDatabaseName "<NewDatabaseName>" –ResourceId $Database.ResourceID

# Verify the status of restored database
$RestoredDatabase.status

```

>[AZURE.NOTE] サーバーが foo.database.windows.net の場合、上記の PowerShell コマンドレットで -ServerName として "foo" を使用します。


### REST API
プログラムでデータベースの復元を実行するには、REST を使用します。

1. 「データベース復元ポイントの取得」操作を使用して、データベースの復元ポイントの一覧を取得します。
2. 「[データベースの復元要求の作成][]」の操作を使用して、復元を開始します。
3. 「[データベース操作の状態][]」の操作を使用して、復元の状態を追跡します。


>[AZURE.NOTE] 復元が完了したら、「[復旧された Azure SQL データベースの最終処理を行う][]」のガイドに従って、復旧されたデータベースを構成できます。


## 削除されたデータベースの復旧
Azure SQL Data Warehouse では、データベースが削除される前にデータベース スナップショットを取得し、7 日間保持します。データベースが誤って削除された場合、削除されたデータベースを削除された時点の状態に復元することができます。


### Azure ポータル

削除されたデータベースを Azure ポータルを使用して復元するには、次の手順に従います。

1. [Azure ポータル][]にログインします。
2. 画面の左側にある **[参照]** をクリックし、**[SQL Server]** を選択します。
3. 目的のサーバーに移動して選択します。
4. サーバーのブレードで下にスクロールして [操作] を表示し、**[削除済みデータベース]** タイルをクリックします。
5. 復元する削除済みデータベースを選択します。
5. 新しい**データベース名**を指定し、**[作成]** をクリックします。
6. データベースの復元処理が開始され、**[通知]** を使用して処理を監視することができます。


### PowerShell
Azure PowerShell を使用して、削除済みデータベースの復元をプログラムで実行します。Azure PowerShell モジュールをダウンロードするには、[Microsoft Web Platform Installer](http://go.microsoft.com/fwlink/p/?linkid=320376&clcid=0x409) を実行します。Get-Module -ListAvailable -Name AzureRM.Sql を実行することで、バージョンを確認できます。この記事は、Microsoft AzureRM.Sql PowerShell バージョン 1.0.5 に基づいています。

削除済みデータベースを復元するには、[Restore-AzureRmSqlDatabase][] コマンドレットを使用します。

1. Windows PowerShell を開きます。
2. Azure アカウントに接続して、アカウントに関連付けられているすべてのサブスクリプションを一覧表示します。
3. 復元する削除済みデータベースを含むサブスクリプションを選択します。
4. 特定の削除済みデータベースを取得します。
5. その削除済みデータベースを復元します。
6. 復元されたデータベースがオンラインであることを確認します。

```Powershell

Login-AzureRmAccount
Get-AzureRmSubscription
Select-AzureRmSubscription -SubscriptionName "<Subscription_name>"

# Get the deleted database to restore
$DeletedDatabase = Get-AzureRmSqlDeletedDatabaseBackup -ResourceGroupName "<YourResourceGroupName>" -ServerName "<YourServerName>" -DatabaseName "<YourDatabaseName>"

# Restore deleted database
$RestoredDatabase = Restore-AzureRmSqlDatabase –FromDeletedDatabaseBackup –DeletionDate $DeletedDatabase.DeletionDate -ResourceGroupName $DeletedDatabase.ResourceGroupName -ServerName $DeletedDatabase.ServerName -TargetDatabaseName "<NewDatabaseName>" –ResourceId $DeletedDatabase.ResourceID

# Verify the status of restored database
$RestoredDatabase.status

```

>[AZURE.NOTE] サーバーが foo.database.windows.net の場合、上記の PowerShell コマンドレットで -ServerName として "foo" を使用します。


### REST API
プログラムでデータベースの復元を実行するには、REST を使用します。

1.	「[削除された復元可能なデータベースの一覧表示][]」の操作を使用して、復元可能なすべての削除済みデータベースのリストを表示します。
2.	「[削除された復元可能なデータベースの取得][]」の操作を使用して、復元する削除済みデータベースの詳細を取得します。
3.	「[データベースの復元要求の作成][]」の操作を使用して、復元を開始します。
4.	「[データベース操作の状態][]」の操作を使用して、復元の状態を追跡します。


>[AZURE.NOTE] 復元が完了したら、「[復旧された Azure SQL データベースの最終処理を行う][]」のガイドに従って、復旧されたデータベースを構成できます。


## 次のステップ
Azure SQL Database の各エディションのビジネス継続性機能については、「[概要: SQL Database を使用したクラウド ビジネス継続性とデータベース障害復旧][]」をご覧ください。


<!--Image references-->

<!--Article references-->
[概要: SQL Database を使用したクラウド ビジネス継続性とデータベース障害復旧]: sql-database/sql-database-business-continuity.md
[復旧された Azure SQL データベースの最終処理を行う]: sql-database/sql-database-recovered-finalize.md

<!--MSDN references-->
[データベースの復元要求の作成]: http://msdn.microsoft.com/library/azure/dn509571.aspx
[データベース操作の状態]: http://msdn.microsoft.com/library/azure/dn720371.aspx
[削除された復元可能なデータベースの取得]: http://msdn.microsoft.com/library/azure/dn509574.aspx
[削除された復元可能なデータベースの一覧表示]: http://msdn.microsoft.com/library/azure/dn509562.aspx
[Restore-AzureRmSqlDatabase]: https://msdn.microsoft.com/library/mt693390.aspx

<!--Other Web references-->
[Azure ポータル]: https://portal.azure.com/
[サポートに連絡]: https://azure.microsoft.com/blog/azure-limits-quotas-increase-requests/

<!---HONumber=AcomDC_0406_2016-->