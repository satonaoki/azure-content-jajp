<properties
   pageTitle="TSQL で SQL Data Warehouse を作成する |Microsoft Azure"
   description="TSQL で Azure SQL Data Warehouse を作成する方法を説明します。"
   services="sql-data-warehouse"
   documentationCenter="NA"
   authors="lodipalm"
   manager="barbkess"
   editor=""
   tags="azure-sql-data-warehouse"/>

<tags
   ms.service="sql-data-warehouse"
   ms.devlang="NA"
   ms.topic="hero-article"
   ms.tgt_pltfrm="NA"
   ms.workload="data-services"
   ms.date="03/23/2016"
   ms.author="lodipalm;barbkess;sonyama"/>

# Transact-SQL (TSQL) を使用して SQL Data Warehouse データベースを作成する

> [AZURE.SELECTOR]
- [Azure ポータル](sql-data-warehouse-get-started-provision.md)
- [TSQL](sql-data-warehouse-get-started-create-database-tsql.md)
- [PowerShell](sql-data-warehouse-get-started-provision-powershell.md)

この記事では、Transact-SQL (TSQL) を使用して SQL Data Warehouse データベースを作成する方法を示します。

## 開始する前に

この記事の手順を完了するには、次のものが必要です。

- Azure サブスクリプション。Azure サブスクリプションをお持ちでない場合、このページの上部の**無料試用版**をクリックしてからこの記事に戻り、最後まで完了してください。
- 見ることができます。Visual Studio の無償版については、[Visual Studio のダウンロード](https://www.visualstudio.com/downloads/download-visual-studio-vs)に関するページを参照してください。
- V12 論理 SQL サーバー。SQL Data Warehouse を作成するには、V12 SQL サーバーが必要です。V12 論理 SQL サーバーがない場合は、[Azure ポータルのチュートリアル][]で作成方法を参照してください。

## Visual Studio でデータベースを作成する

この記事では、Visual Studio を正しく設定して接続する方法については説明していません。その方法の詳細については、[接続とクエリ][]に関するドキュメントを参照してください。開始するには、Visual Studio で SQL Server オブジェクト エクスプローラーを開き、SQL Data Warehouse データベースの作成に使用するサーバーに接続します。それが完了したら、Master データベースに対して次のコマンドを実行することで、SQL Data Warehouse を作成できます。

```sql
CREATE DATABASE <Name> (EDITION='datawarehouse', SERVICE_OBJECTIVE = '<Compute Size - DW####>', MAXSIZE= <Storage Size - #### GB>);
```

## sqlcmd でデータベースを作成する

コマンド ラインを開き、次のコマンドを実行して SQL Data Warehouse を作成することもできます。

```sql
sqlcmd -S <Server Name>.database.windows.net -I -U <User> -P <Password> -Q "CREATE DATABASE <Name> (EDITION='datawarehouse', SERVICE_OBJECTIVE = '<Compute Size - DW####>', MAXSIZE= <Storage Size - #### GB>)"
```

上記の TSQL ステートメントを実行する際は、`MAXSIZE` パラメーターと `SERVICE_OBJECTIVE` パラメーターに注意してください。これらは、Data Warehouse インスタンスに割り当てられる最初のストレージ サイズおよびコンピューティングを決定します。`MAXSIZE` には次のいずれかのサイズを指定できますが、将来の拡張を見越して、より大きなサイズを選択することをお勧めします。2

+ 50 GB
+ 500 GB
+ 750 GB
+ 1,024 GB
+ 5,120 GB
+ 10,240 GB
+ 20,480 GB
+ 30,720 GB
+ 40,960 GB
+ 51,200 GB

`SERVICE_OBJECTIVE` は、インスタンスに最初に割り当てられる DWU の数を示し、次のいずれかの値を指定できます。

+ DW100
+ DW200
+ DW300
+ DW400
+ DW500
+ DW600
+ DW1000
+ DW1200
+ DW1500
+ DW2000

これらのパラメーターと課金の関係については、[価格のページ][]を参照してください。

## 次のステップ
SQL Data Warehouse のプロビジョニングが済めば、[サンプル データを読み込んだり][]、[開発][]、[読み込み][]、[移行][]の方法を調べたりできます。

[Azure ポータルのチュートリアル]: ./sql-data-warehouse-get-started-provision.md
[接続とクエリ]: ./sql-data-warehouse-get-started-connect.md
[移行]: ./sql-data-warehouse-overview-migrate.md
[開発]: ./sql-data-warehouse-overview-develop.md
[読み込み]: ./sql-data-warehouse-overview-load.md
[サンプル データを読み込んだり]: ./sql-data-warehouse-get-started-manually-load-samples.md
[価格のページ]: https://azure.microsoft.com/pricing/details/sql-data-warehouse/

<!---HONumber=AcomDC_0330_2016-->