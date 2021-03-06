<properties 
	pageTitle="Azure Storage への接続文字列の構成 |Microsoft Azure"
	description="Azure ストレージ アカウントへの接続文字列を作成します。接続文字列には、アプリケーションからストレージ アカウント内のリソースへのアクセスを認証するために必要な情報が含まれています。"
	services="storage"
	documentationCenter=""
	authors="tamram"
	manager="carmonm"
	editor="tysonn"/>

<tags
	ms.service="storage"
	ms.workload="storage"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="04/01/2016"
	ms.author="tamram"/>

# Azure Storage の接続文字列を構成する

## 概要

接続文字列には、Azure Storage リソースにプログラムでアクセスするのに必要な情報が含まれています。アプリケーションは、接続文字列を使用して、アクセスの認証に必要な情報を Azure Storage に提供します。

接続文字列は、次の目的に構成できます。

- サービスやアプリケーションをローカルでテストしているときに、Azure ストレージ エミュレーターに接続します。
- ストレージ サービスの既定のエンドポイントまたはユーザーが定義した明示的なエンドポイントを使用して、Azure のストレージ アカウントに接続します。
- 共有アクセス署名 (SAS) を使用してストレージ アカウント内のリソースにアクセスします。

## 接続文字列の保存

アプリケーションは実行時に Azure Storage へのアクセスを認証するために、接続文字列を保存する必要があります。接続文字列の保存にはいくつかのオプションがあります。

- デスクトップまたはデバイスで実行するアプリケーションの場合は、app.config ファイルまたは別の構成ファイルに接続文字列を保存できます。app.config ファイルを使用する場合は、接続文字列を **AppSettings** セクションに追加します。
- アプリケーションが Azure クラウド サービスで実行している場合、接続文字列は [Azure サービスの構成スキーマ (.cscfg) ファイル](https://msdn.microsoft.com/library/ee758710.aspx)に保存できます。接続文字列をサービス構成ファイルの **ConfigurationSettings** セクションに追加します。

構成ファイル内に接続文字列を保存すると、接続文字列を簡単に更新してストレージ エミュレーターとクラウドの Azure ストレージ アカウントを切り替えることができます。接続文字列を編集するだけでストレージ アカウントを参照できます。

[Microsoft Azure Configuration Manager](https://www.nuget.org/packages/Microsoft.WindowsAzure.ConfigurationManager/) クラスを使用すると、アプリケーションの実行場所に関係なく、実行時に接続文字列にアクセスできます。

## ストレージ エミュレーターへの接続文字列を作成する

[AZURE.INCLUDE [storage-emulator-connection-string-include](../../includes/storage-emulator-connection-string-include.md)]

ストレージ エミュレーターについて詳しくは、「[開発とテストのための Azure のストレージ エミュレーター使用](storage-use-emulator.md)」を参照してください。

## Azure ストレージ アカウントへの接続文字列を作成する

Azure ストレージ アカウントへの接続文字列を作成するには、次の接続文字列の形式を使用します。HTTP と HTTPS (推奨) のどちらを使用してストレージ アカウントに接続するかを指定し、`myAccountName` をストレージ アカウントの名前に、`myAccountKey` をアカウント アクセス キーに置き換えます。

    DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey

たとえば、接続文字列は次のサンプルの接続文字列のようになります。

	DefaultEndpointsProtocol=https;
	AccountName=storagesample;
	AccountKey=<account-key>

> [AZURE.NOTE] Azure Storage では、HTTP と HTTPS の両方の接続文字列をサポートします。ただし、HTTPS の使用を強くお勧めします。

## 明示的なストレージ エンドポイントへの接続文字列を作成する

次の場合は、接続文字列でサービス エンドポイントを明示的に指定することができます。

- ストレージ アカウントのカスタム ドメイン名を BLOB サービスとマップしている場合。
- ストレージ アカウント内のストレージ リソースにアクセスするための共有アクセス署名がある場合。

明示的な BLOB エンドポイントを指定する接続文字列を作成するには、次に示す形式を使用して、プロトコル仕様 (HTTP または HTTPS) を含む完全なサービス エンドポイントをサービスごとに指定します。

	BlobEndpoint=myBlobEndpoint;
	QueueEndpoint=myQueueEndpoint;
	TableEndpoint=myTableEndpoint;
	FileEndpoint=myFileEndpoint;
	[credentials]


サービス エンドポイントは 1 つ以上指定する必要がありますが、すべてを指定する必要はありません。たとえば、カスタムの BLOB エンドポイントで使用する接続文字列を作成する場合は、キューとテーブルのエンドポイントの指定は省略できます。接続文字列からキューとテーブルのエンドポイントを除外することを選択した場合、その接続文字列を使用してコードからキュー サービスやテーブル サービスにアクセスすることはできないことにご注意ください。

接続文字列でサービス エンドポイントを明示的に指定する場合、上記の文字列に `credentials` を指定するには、次の 2 つのオプションがあります。

- アカウント名とキーを指定する: `AccountName=myAccountName;AccountKey=myAccountKey`
- 共有アクセス署名を指定する: `SharedAccessSignature=base64Signature`

### カスタム ドメイン名を使用して BLOB エンドポイントを指定する

BLOB サービスで使用するためにカスタム ドメイン名が登録されている場合は、接続文字列で BLOB エンドポイントを明示的に構成する必要があります。接続文字列に示されているエンドポイント値を使用することで、BLOB サービスへの要求 URI が作成され、コードに返される URI の形式も決定されます。

たとえば、カスタム ドメイン上の BLOB エンドポイントへの接続文字列は、次のようになります。

	DefaultEndpointsProtocol=https;
	BlobEndpoint=www.mydomain.com;
	AccountName=storagesample;
	AccountKey=<account-key>


### 共有アクセス署名で BLOB エンドポイントを指定する

明示的なエンドポイントを持つ接続文字列を作成し、共有アクセス署名を介してストレージ リソースにアクセスできます。この場合、アカウント名とアカウント キーの資格情報ではなく、共有アクセス署名を接続文字列に含めて指定できます。共有アクセス署名トークンは、アクセス先のリソース、アクセス可能な期間、付与される権限に関する情報をカプセル化します。共有アクセス署名の詳細については、「[Shared Access Signature、第 1 部: SAS モデルについて](storage-dotnet-shared-access-signature-part-1.md)」を参照してください。

共有アクセス署名を含む接続文字列を作成するには、文字列を次の形式で指定します。

    BlobEndpoint=myBlobEndpoint; QueueEndpoint=myQueueEndpoint;TableEndpoint=myTableEndpoint;SharedAccessSignature=sasToken

エンドポイントには、既定のサービス エンドポイントとカスタム エンドポイントのいずれかを指定できます。`sasToken` は、SAS URL の疑問符 (?) に続くクエリ文字列です。

### エンドポイント サフィックスを含む接続文字列を作成する

Azure China、Azure Governance など、別のエンドポイント サフィックスを持つリージョンまたはインスタンスのストレージ サービスの接続文字列を作成するには、次の接続文字列の形式を使用します。HTTP または HTTPS のどちらでストレージ アカウントに接続するかを示し、`myAccountName` をストレージ アカウントの名前、`myAccountKey` をアカウント アクセス キー、`mySuffix` を URI サフィックスに置き換えます。


	DefaultEndpointsProtocol=[http|https];AccountName=myAccountName;AccountKey=myAccountKey;EndpointSuffix=mySuffix;


たとえば、接続文字列は次のサンプルの接続文字列のように表示されます。

	DefaultEndpointsProtocol=https;
	AccountName=storagesample;
	AccountKey=<account-key>;
	EndpointSuffix=core.chinacloudapi.cn;

## 次のステップ

- [開発とテストのための Azure のストレージ エミュレーター使用](storage-use-emulator.md)
- [Azure ストレージ エクスプローラー](storage-explorers.md)

<!---HONumber=AcomDC_0406_2016-->