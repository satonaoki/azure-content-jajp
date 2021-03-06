<properties
   pageTitle="Azure Automation のセキュリティ"
   description="この記事では、Azure Automation の Automation アカウントで使用できるセキュリティとさまざまな認証方法の概要を説明します。"
   services="automation"
   documentationCenter=""
   authors="MGoedtel"
   manager="jwhit"
   editor="tysonn" />
<tags
   ms.service="automation"
   ms.devlang="na"
   ms.topic="get-started-article"
   ms.tgt_pltfrm="na"
   ms.workload="infrastructure-services"
   ms.date="04/08/2016"
   ms.author="magoedte" />

# Azure Automation のセキュリティ
Azure Automation を使用すると、Azure 内のリソース、オンプレミスのリソース、Amazon Web Services (AWS) などの他のクラウド プロバイダーのリソースに対するタスクを自動化できます。Runbook が必要な操作を実行するためには、操作対象のリソースに安全に、サブスクリプション内で必要な最低限の権限だけでアクセスするための、アクセス許可が必要です。この記事では、Azure Automation でサポートされるさまざまなセキュリティ シナリオについて説明し、管理する必要のある環境に基づいて開始する方法を示します。

## Automation アカウントの概要
Azure Automation を初めて開始するときに、少なくとも 1 つの Automation アカウントを作成する必要があります。Automation アカウントを使用すると、Automation アカウントごとに Automation リソース (Runbook、資産、構成) を分離できます。Automation アカウントを使用してリソースを異なる論理環境に分けることができます。たとえば、開発環境用、本番環境用、オンプレミス環境用に、それぞれ異なるアカウントを使用できます。Azure Automation アカウントは、Microsoft アカウントや、Azure サブスクリプションで作成されたアカウントとは異なります。

各 Automation アカウントの Automation リソースは単一の Azure リージョンと関連付けられていますが、Automation アカウントではあらゆるリージョンのリソースを管理できます。異なるリージョンで Automation アカウントを作成する主な理由としては、特定のリージョンに分離しなければならないデータやリソースを必要とするポリシーがある場合が挙げられます。

>[AZURE.NOTE]Azure ポータルで作成した Automation アカウントおよび各アカウントに含まれるリソースには、Azure クラシック ポータルでアクセスすることはできません。これらのアカウントとリソースを Windows PowerShell で管理するには、[Azure リソース マネージャー] モジュールを使用する必要があります。

Azure Automation で Azure Resource Manager (ARM) と Azure コマンドレットを使用してリソースに対して実行するすべてのタスクは、Azure Active Directory の組織 ID 資格情報に基づく認証を使用して、Azure に対する認証を行う必要があります。Azure サービス管理 (ASM) モードではもともと証明書ベースの認証方法が使用されていましたが、セットアップが複雑でした。Azure AD による Azure の認証は 2014 年に導入されましたが、これは、認証アカウントの構成処理を簡単にするだけでなく、ASM と ARM モードの両方で使用できる単一のユーザー アカウントによる Azure の非対話形式の認証機能をサポートするためでもありました。

最近リリースされた別の更新により、Automation アカウントを作成するときに Azure AD サービス プリンシパル オブジェクトを自動的に作成できるようになりました。これは Azure 実行アカウントと呼ばれ、Azure Resource Manager での Runbook 自動化の既定の認証方法です。

ロール ベースのアクセス制御は、Azure AD ユーザー アカウントおよびサービス プリンシパルに対して許可されたアクションを付与し、そのサービス プリンシパルを認証するために、ARM モードで使用できます。Automation アクセス許可を管理するためのモデルの開発に関する詳細については、「[Azure Automation におけるロールベースのアクセス制御](../automation/automation-role-based-access-control.md)」を参照してください。

データセンター内の Hybrid Runbook Worker で、または AWS のコンピューティング サービスに対して実行されている Runbook は、Runbook が Azure のリソースを認証するために通常使用される方法と同じものを使用することはできません。これは、これらのリソースは Azure の外部で実行しており、そのため、ローカルにアクセスするリソースに対する認証を行うには Automation で定義されている個別のセキュリティ資格情報を必要とするためです。

## 認証方法

次の表は、Azure Automation によってサポートされる各環境のさまざまな認証方法と、Runbook 用の認証のセットアップ方法に関する記事をまとめたものです。

メソッド | 環境 | 記事
----------|----------|----------
Azure AD ユーザー アカウント | Azure リソース マネージャーと Azure サービス管理 | [Authenticate Runbooks with Azure AD User account (Azure AD ユーザー アカウントでの Runbook の認証)](../automation/automation-sec-configure-aduser-account.md)
Azure AD サービス プリンシパル オブジェクト | Azure リソース マネージャー | [Azure 実行アカウントを使用した Runbook の認証](../automation/automation-sec-configure-azure-runas-account.md)
Windows 認証 | オンプレミスのデータセンター | [Azure Automation の Hybrid Runbook Worker](../automation/automation-hybrid-runbook-worker.md)
AWS 資格情報 | Amazon Web Services | [Authenticate Runbooks with Amazon Web Services (AWS) (Amazon Web Services (AWS) での Runbook の認証)](../automation/automation-sec-configure-aws-account.md)

<!---HONumber=AcomDC_0413_2016-->