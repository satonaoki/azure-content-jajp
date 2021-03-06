<properties
	pageTitle="Azure Insights: 自動スケール操作を使用して電子メールと Webhook アラート通知を送信する | Microsoft Azure"
	description="Azure Insights で自動スケール操作を使用して Web URL を呼び出したり、電子メール通知を送信したりする方法について説明します。"
	authors="kamathashwin"
	manager=""
	editor=""
	services="azure-portal"
	documentationCenter="na"/>

<tags
	ms.service="azure-portal"
	ms.workload="na"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="03/30/2016"
	ms.author="ashwink"/>

# Azure Insights で自動スケール操作を使用して電子メールと Webhook アラート通知を送信する

この記事では、Azure で自動スケール操作に基づいて特定の Web URL を呼び出すことや電子メールを送信することができるようにトリガーを設定する方法について説明します。

## Webhook
Webhook を使用すると、後処理やカスタム通知のために、Azure アラート通知を他のシステムにルーティングすることができます。たとえば、受信 Web 要求を処理して SMS を送信する、バグをログに記録する、チャットやメッセージング サービスを使用してチームに通知するなどのサービスにアラートをルーティングすることができます。Webhook URI は有効な HTTP または HTTPS エンドポイントである必要があります。

## 電子メール
電子メールは、任意の有効な電子メール アドレスに送信できます。このルールが実行されているサブスクリプションの管理者と共同管理者にも通知されます。


## Cloud Services と Web Apps
Azure ポータルから Cloud Services とサーバー ファーム (Web Apps) をオプトインできます。

- **[スケールの基準]** メトリックを選択します。

![scale by](./media/insights-autoscale-to-webhook-email/insights-autoscale-scale-by.png)

## 仮想マシン スケール セット
新しい ARM ベースの仮想マシン (仮想マシン スケール セット) の場合は、REST API、PowerShell、および CLI を使用して構成できます。ポータルのインターフェイスはまだ使用できません。


## Webhook での認証
認証 URI には、次の 2 つの形式があります。

	1. Token-base authentication, where you save the webhook URI with a token ID as a query parameter. For example, https://mysamplealert/webcallback?tokenid=sometokenid&someparameter=somevalue
	2. Basic authentication, where you use a user ID and password. For example, https://userid:password@mysamplealert/webcallback?someparamater=somevalue&parameter=value

## 自動スケール通知の Webhook ペイロード スキーマ
自動スケール通知が生成されると、次のメタデータが Webhook ペイロードに含まれます。

```
{
        "version": "1.0",
        "status": "Activated",
        "operation": "Scale In",
        "context": {
                "timestamp": "2016-03-11T07:31:04.5834118Z",
                "id": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.insights/autoscalesettings/myautoscaleSetting",
                "name": "myautoscaleSetting",
                "details": "Autoscale successfully started scale operation for resource 'MyCSRole' from capacity '3' to capacity '2'",
                "subscriptionId": "s1",
                "resourceGroupName": "rg1",
                "resourceName": "MyCSRole",
                "resourceType": "microsoft.classiccompute/domainnames/slots/roles",
                "resourceId": "/subscriptions/s1/resourceGroups/rg1/providers/microsoft.classicCompute/domainNames/myCloudService/slots/Production/roles/MyCSRole",
                "portalLink": "https://portal.azure.com/#resource/subscriptions/s1/resourceGroups/rg1/providers/microsoft.classicCompute/domainNames/myCloudService",
                "oldCapacity": "3",
                "newCapacity": "2"
        },
        "properties": {
                "key1": "value1",
                "key2": "value2"
        }
}
```


|フィールド |必須|	説明|
|---|---|---|
|status |○ |自動スケール操作が生成されたことを示す状態。|
|operation|	○ |インスタンスの増加の場合は "Scale Out"、インスタンスの減少の場合は "Scale In" になります。|
|context|	○ |自動スケール操作のコンテキスト。|
|timestamp|	○ |自動スケール操作がトリガーされたときのタイム スタンプ。|
|id |あり|	自動スケール設定の ARM (Azure Resource Manager) ID。|
|name |あり|	自動スケール設定の名前。|
|詳細|	あり |自動スケール サービスが実行した操作とインスタンス数の変更の説明。|
|subscriptionId|	あり |スケールされるターゲット リソースのサブスクリプション ID。|
|resourceGroupName|	あり|	スケールされるターゲット リソースのリソース グループ名。|
|resourceName |あり|	スケールされるターゲット リソースの名前。|
|resourceType |あり|	次の 3 つの値がサポートされています: "microsoft.classiccompute/domainnames/slots/roles" (クラウド サービス ロール)、"microsoft.compute/virtualmachinescalesets" (仮想マシン スケール セット)、"Microsoft.Web/serverfarms" (Web アプリ)。|
|resourceId |あり|スケールされるターゲット リソースの ARM ID。|
|portalLink |あり |ターゲット リソースの概要ページへの Azure ポータルのリンク。|
|oldCapacity|	あり |自動スケールによってスケール操作が実行された時点の (以前の) インスタンス数。|
|newCapacity|	あり |自動スケールによってリソースがスケールされた後の新しいインスタンス数。|
|プロパティ|	いいえ|	省略可能。<Key  Value> ペアのセット (例: Dictionary <String  String>)。properties フィールドは省略可能です。カスタム ユーザー インターフェイスまたはロジック アプリ ベースのワークフローでは、ペイロードを使用して渡すことのできるキーと値を入力できます。Webhook URI 自体を (クエリ パラメーターとして) 使用して、カスタム プロパティを送信 Webhook 呼び出しに戻すこともできます。|

<!---HONumber=AcomDC_0330_2016------>