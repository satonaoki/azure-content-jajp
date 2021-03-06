<properties
	pageTitle="Azure VM への SSH 接続のトラブルシューティング | Microsoft Azure"
	description="Linux を実行する Azure 仮想マシンに対する SSH 接続の失敗や拒否などの SSH エラーをトラブルシューティングして修正します。"
	keywords="拒否されたSSH 接続,SSH エラー,Azure SSH,失敗した SSH 接続"
	services="virtual-machines-linux"
	documentationCenter=""
	authors="dsk-2015"
	manager="timlt"
	editor=""
	tags="top-support-issue,azure-service-management,azure-resource-manager"/>

<tags
	ms.service="virtual-machines-linux"
	ms.workload="infrastructure-services"
	ms.tgt_pltfrm="vm-linux"
	ms.devlang="na"
	ms.topic="article"
	ms.date="01/22/2016"
	ms.author="dkshir"/>

# Linux ベースの Azure 仮想マシンに対する Secure Shell (SSH) 接続のトラブルシューティング

Linux ベースの Azure 仮想マシンに接続しようとしたときに発生する SSH エラーには、さまざまな原因が考えられます。この記事は、そのような原因の特定と修正について説明します。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-both-include.md)]

この記事は、Linux を実行する Azure 仮想マシンにのみ適用されます。Windows を実行する Azure Virtual Machines の場合は、「[Azure VM へのリモート デスクトップ接続のトラブルシューティング](virtual-machines-windows-troubleshoot-rdp-connection.md)」を参照してください。

この記事についてさらにヘルプが必要な場合は、いつでも [MSDN の Azure フォーラムとスタック オーバーフロー フォーラム](http://azure.microsoft.com/support/forums/)で Azure エキスパートに問い合わせることができます。または、Azure サポート インシデントを送信できます。その場合は、[Azure サポートのサイト](http://azure.microsoft.com/support/options/)に移動して、**[サポートの要求]** をクリックします。Azure サポートの使用方法の詳細については、「[Azure Support FAQ (Microsoft Azure サポートに関する FAQ)](http://azure.microsoft.com/support/faq/)」を参照してください。


## 一般的な SSH エラーの修正

このセクションでは、一般的な SSH 接続の問題をすばやく修正するための手順を示します。

### クラシック デプロイ モデルを使用して作成した仮想マシン

最も一般的な SSH 接続エラーを解決するには、次の手順を試します。

1. [Azure ポータル](https://portal.azure.com)から_リモート アクセスをリセット_します。<br> **[参照]**、**[仮想マシン (クラシック)]**、ご使用の Linux 仮想マシン、**[リモートのリセット]** の順にクリックします。

2. 仮想マシンを再起動します。<br> [Azure ポータル](https://portal.azure.com)から、**[参照]**、**[仮想マシン (クラシック)]**、ご使用の Linux 仮想マシン、**[再起動]** の順にクリックします。<br> [Azure クラシック ポータル](https://manage.windowsazure.com)で、**[仮想マシン]**、**[インスタンス]** の順に開いて、**[再起動]** をクリックします。

3. [仮想マシンのサイズを変更します。](https://msdn.microsoft.com/library/dn168976.aspx)

4. 「[Linux 仮想マシンのパスワードまたは SSH をリセットする方法](virtual-machines-linux-classic-reset-access.md)」の指示に従って、仮想マシンで次の操作を行います。

	- パスワードまたは SSH キーをリセットする
	- 新しい _sudo_ ユーザー アカウントを作成する
	- SSH 構成をリセットする

5. プラットフォームの問題の有無について VM のリソースの状態を確認します。<br> **[参照]**、**[仮想マシン (クラシック)]**、ご使用の Linux 仮想マシン、**[設定]**、**[正常性の確認]** の順にクリックします。


### リソース マネージャー デプロイ モデルを使用して作成した仮想マシン

リソース マネージャーのデプロイ モデルを使用して作成された仮想マシンの一般的な SSH の問題を解決するには、次の手順を試してください。

1. Azure CLI または Azure PowerShell のいずれかを使用し、コマンド ラインで Linux VM への _SSH 接続をリセット_します。[Microsoft Azure Linux エージェント](virtual-machines-linux-agent-user-guide.md)のバージョン 2.0.5 以降がインストールされていることを確認します。

**Azure CLI の使用**:

a.まだインストールしていない場合は、[Azure CLI をインストールし](../xplat-cli-install.md)、`azure login` コマンドを使用して Azure サブスクリプションに接続します。

b.リソース マネージャー モードになっていることを確認します。最新バージョンの Azure CLI は、既定でリソース マネージャー モードになっています。

	```
	azure config mode arm
	```

c.次の方法のいずれかを使用して SSH 接続をリセットします。

* 次の例のように、`vm reset-access` コマンドを使用します。

	```
	azure vm reset-access -g YourResourceGroupName -n YourVirtualMachineName -r
	```

これにより、`VMAccessForLinux` 拡張機能が仮想マシンにインストールされます。

* あるいは、次の内容を含む PrivateConf.json という名前のファイルを作成します。

	```
	{  
		"reset_ssh":"True"
	}
	```

その後、`VMAccessForLinux` 拡張機能を手動で実行して SSH 接続をリセットします。

	```
	azure vm extension set "YourResourceGroupName" "YourVirtualMachineName" VMAccessForLinux Microsoft.OSTCExtensions "1.2" --private-config-path PrivateConf.json
	```

**Azure PowerShell の使用**:

a.まだインストールしていない場合は、[Azure PowerShell をインストールし、Azure AD メソッドを使用して Azure サブスクリプションに接続します](../powershell-install-configure.md)。Azure PowerShell の 1.0.x より前のバージョンでは、_Switch-AzureMode_ を使用して、明示的にリソース マネージャー モードに切り替える必要があります。

b.次の例のように、`VMAccessForLinux` 拡張機能を実行して SSH 接続をリセットします。以前のバージョンでは、コマンドは _Set-AzureVMExtension_ です。

	```
	Set-AzureRmVMExtension -ResourceGroupName "yourRG" -VMName "yourVM" -Location "West US" -Name "VMAccessForLinux" -Publisher "Microsoft.OSTCExtensions" -ExtensionType "VMAccessForLinux" -TypeHandlerVersion "1.2" -SettingString "{}" -ProtectedSettingString '{"reset_ssh":true}'
	```

2. [Azure ポータル](https://portal.azure.com)から Linux VM を再起動します。<br> **[参照]**、**[仮想マシン]**、ご使用の Linux 仮想マシン、**[再起動]** の順にクリックします。

3. Azure CLI または Azure PowerShell のいずれかを使用し、コマンド ラインで Linux VM の _パスワードまたは SSH キーをリセット_します。次の例のように、_sudo_ 権限を持つ新しいユーザー名とパスワードを作成することもできます。

**Azure CLI の使用**:

既に説明したように Azure CLI をインストールして構成します。必要に応じてリソース マネージャー モードに切り替えてから、次のいずれかの方法で拡張機能を実行します。

* `vm reset-access` コマンドを実行して任意の SSH 資格情報を設定します。

	```
	azure vm reset-access TestRgV2 TestVmV2 -u NewUser -p NewPassword
	```

コマンド ラインで `azure vm reset-access -h` を入力すると、このコマンドの詳細が表示されます。

* または、次の内容を含む PrivateConf.json という名前のファイルを作成します。

	```
	{
		"username":"NewUsername", "password":"NewPassword", "expiration":"2016-01-01", "ssh_key":"", "reset_ssh":false, "remove_user":""
	}
	```

上記のファイルを使用して Linux 拡張機能を実行します。

	```
	$azure vm extension set "testRG" "testVM" VMAccessForLinux Microsoft.OSTCExtensions "1.2" --private-config-path PrivateConf.json
	```

「[Linux 仮想マシンのパスワードまたは SSH をリセットする方法](virtual-machines-linux-classic-reset-access.md)」と同様の手順に従って、他のバリエーションを試すこともできます。必ずリソース マネージャー モード用に Azure CLI の命令を変更してください。


**Azure PowerShell の使用**:

既に説明したように Azure PowerShell をインストールして構成します。リソース マネージャー モードに切り替えてから、次のように拡張機能を実行します。

	```
	$RGName = 'testRG'
	$VmName = 'testVM'
	$Location = 'West US'

	$ExtensionName = 'VMAccessForLinux'
	$Publisher = 'Microsoft.OSTCExtensions'
	$Version = '1.2'

	$PublicConf = '{}'
	$PrivateConf = '{"username":"NewUsername", "password":"NewPassword", "ssh_key":"", "reset_ssh":false, "remove_user":""}'

	Set-AzureRmVMExtension -ResourceGroupName $RGName -VMName $VmName -Location $Location -Name $ExtensionName -Publisher $Publisher -ExtensionType $ExtensionName -TypeHandlerVersion $Version -SettingString $PublicConf -ProtectedSettingString $PrivateConf
	```

必ず $RGName、$VmName、$Location、SSH 資格情報の値を、インストールに固有の値で置き換えてください。



## SSH エラーの詳細なトラブルシューティング

それでも SSH クライアントが仮想マシン上の SSH サービスに到達できない場合は、いくつかの理由が考えられます。関連のあるコンポーネントを次に示します。

![SSH サービスのコンポーネントを示す図](./media/virtual-machines-linux-troubleshoot-ssh-connection/ssh-tshoot1.png)

次のセクションは、エラーのソースを独立させて、解決策と回避策を見つけ出すのに役立ちます。

### 準備作業

まず、ポータルで仮想マシンの状態を確認します。

[Azure クラシック ポータル](https://manage.windowsazure.com)で、クラシック デプロイ モデルの仮想マシンについて、次の手順を実行します。

1. **[仮想マシン]** > *VM 名*をクリックします。
2. VM の**ダッシュボード**をクリックし、その状態を確認します。
3. コンピューティング、記憶域、およびネットワーク リソースの最近のアクティビティを確認するには、**[監視]** をクリックします。
4. SSH トラフィックのエンドポイントがあることを確認するには、**[エンドポイント]** をクリックします。

[Azure ポータル](https://portal.azure.com)で次の操作を行います。

1. クラシック デプロイ モデルで作成された仮想マシンの場合は、**[参照]**、**[仮想マシン (クラシック)]**、*VM 名*の順にクリックします。リソース マネージャーを使用して作成された仮想マシンの場合は、**[参照]**、**[仮想マシン]**、*VM 名*の順にクリックします。仮想マシンの状態ウィンドウには、"**実行中**" と表示されます。コンピューティング、ストレージ、およびネットワーク リソースの最近のアクティビティを確認するには、下にスクロールします。
2. エンドポイント、IP アドレスなどの設定を確認するには、**[設定]** をクリックします。リソース マネージャーを使用して作成された仮想マシンのエンドポイントを特定するには、[ネットワーク セキュリティ グループ](../virtual-network/virtual-networks-nsg.md)が定義されているかどうか、ネットワーク セキュリティ グループにルールが適用されているかどうか、それらのルールがサブネットで参照されているかどうかを確認します。

ネットワーク接続を確認するには、構成されているエンドポイントを確認します。また、HTTP などの別のプロトコルや他のサービスを使用して、VM に到達できるかどうかを確認します。

以上の手順を完了したら、SSH 接続を再試行してください。


### 問題の原因を調べる

お使いのコンピューター上の SSH クライアントは、以下の問題または誤構成が原因で Azure 仮想マシンの SSH サービスに到達できない場合があります。

- SSH クライアント コンピューター
- 組織の境界デバイス
- クラウド サービス エンドポイントとアクセス制御リスト (ACL)
- ネットワーク セキュリティ グループ
- Linux ベースの Azure 仮想マシン

#### ソース 1: SSH クライアント コンピューター

エラーのソースであるコンピューターを排除するには、そのコンピューターから他のオンプレミスの Linux ベース コンピューターに SSH 接続できることを確認します。

![SSH クライアント コンピューターのコンポーネントを示す図](./media/virtual-machines-linux-troubleshoot-ssh-connection/ssh-tshoot2.png)

接続できない場合は、そのコンピューターで以下を確認してください。

- 受信または送信の SSH トラフィック (TCP 22) をブロックしているローカル ファイアウォールの設定
- SSH 接続を妨げている、ローカルでインストールされたクライアント プロキシのソフトウェア
- SSH 接続を妨げている、ローカルでインストールされたネットワーク監視ソフトウェア
- トラフィックを監視する、あるいは特定種類のトラフィックを許可または禁止する別の種類のセキュリティ ソフトウェア

いずれの場合も、該当ソフトウェアを一時的に無効にし、オンプレミス コンピューターへの SSH 接続を試すことで、原因を確認してください。次に、ネットワーク管理者とともに、SSH 接続が可能になるようにそのソフトウェアの設定を修正してください。

証明書認証を使用している場合は、ホーム ディレクトリの .ssh フォルダーに対してこれらのアクセス許可があることを確認します。

- Chmod 700 ~/.ssh
- Chmod 644 ~/.ssh/*.pub
- Chmod 600 ~/.ssh/id\_rsa (またはプライベート キーが格納されている他のファイル)
- Chmod 644 ~/.ssh/known\_hosts (SSH 経由で接続したホストが含まれます)

#### ソース 2: 組織のエッジ デバイス

エラーのソースである組織のエッジ デバイスを排除するには、インターネットに直接接続されているコンピューターから Azure VM に SSH 接続できるかを確認します。サイト間 VPN 接続または ExpressRoute 接続を介して VM にアクセスする場合は、「[ソース 4: ネットワーク セキュリティ グループ](#nsg)」に進みます。

![組織の境界デバイスを示す図](./media/virtual-machines-linux-troubleshoot-ssh-connection/ssh-tshoot3.png)

インターネットに直接接続されているコンピューターがない場合は、新しい Azure 仮想マシンをリソース グループまたはクラウド サービス内に簡単に作成して使用できます。詳細については、「[Azure 上で Linux を実行する仮想マシンの作成](virtual-machines-linux-quick-create-cli.md)」を参照してください。テストの完了後に、そのリソース グループまたは仮想マシンと、クラウド サービスを削除します。

インターネットに直接接続されているコンピューターとの SSH 接続を作成できるのなら、組織のエッジ デバイスで以下を確認してください。

- インターネットでの SSH トラフィックをブロックしている内部ファイアウォール
- SSH 接続を妨げているプロキシ サーバー
- SSH 接続を妨げている、エッジ ネットワーク内のデバイスで実行されている侵入検出またはネットワーク監視のソフトウェア

ネットワーク管理者と協力して、インターネットでの SSH トラフィックを許可するように組織のエッジ デバイスの設定を修正します。

#### ソース 3: クラウド サービス エンドポイントと ACL

> [AZURE.NOTE] このソースは、クラシック デプロイ モデルを使用して作成した仮想マシンのみに適用されます。リソース マネージャーを使用して作成した仮想マシンについては、「[ソース 4: ネットワーク セキュリティ グループ](#nsg)」に進んでください。

エラーの原因であるクラウド サービス エンドポイントと ACL を排除するには、[クラシック デプロイ モデル](../resource-manager-deployment-model.md)を使用して作成した VM の場合、同じ仮想ネットワーク内の別の Azure VM から自身の VM に SSH 接続できるかどうかをチェックします。

![クラウド サービス エンドポイントと ACL を示す図](./media/virtual-machines-linux-troubleshoot-ssh-connection/ssh-tshoot4.png)

同じ仮想ネットワーク内に別の VM がない場合、新しい VM を簡単に作成することができます。詳細については、「[Azure 上で Linux を実行する仮想マシンの作成](virtual-machines-linux-quick-create-cli.md)」を参照してください。テストの完了後に、追加した VM を削除してください。

同じ仮想ネットワーク内にある VM に対して SSH 接続を作成できる場合は、次の点を確認します。

- ターゲットの VM での SSH トラフィック向けエンドポイントの構成。エンドポイントのプライベート TCP ポートは、VM 上の SSH サービスがリッスンする TCP ポートと一致する必要があります (既定値は 22 です)。リソース マネージャーのデプロイ モデルでテンプレートを使用して作成した VM の場合は、Azure ポータルで **[参照]**、**[仮想マシン (v2)]**、*VM 名*、**[設定]**、**[エンドポイント]** の順に選択して、SSH TCP ポート番号を確認します。
- ターゲットの仮想マシンでの、SSH トラフィック向けエンドポイントの ACL。ACL を使用すると、発信元 IP アドレスに基づいて、インターネットからの受信トラフィックを許可または拒否するかを指定できます。ACL が正しく構成されていないと、そのエンドポイントへの SSH 受信トラフィックを受け取れない場合があります。ご利用になっているプロキシのパブリック IP アドレスからの受信トラフィック、または他のエッジ サーバーからの受信トラフィックが許可されているかを ACL で確認してください。詳細については、[ネットワーク アクセス制御リスト (ACL) の概要](../virtual-network/virtual-networks-acl.md)に関するページを参照してください。

問題の原因であるエンドポイントを排除するには、現在のエンドポイントを削除し、新しいエンドポイントを作成して、**SSH** 名を指定します (パブリックとプライベートのポート番号には TCP ポート 22)。詳細については、「[Azure での仮想マシンに対するエンドポイントの設定](virtual-machines-windows-classic-setup-endpoints.md)」をご覧ください

<a id="nsg"></a>
#### ソース 4: ネットワーク セキュリティ グループ

ネットワーク セキュリティ グループでは、許可された受信トラフィックと送信トラフィックをより細かく制御できます。Azure 仮想ネットワーク内のサブネットまたはクラウド サービスの全体に適用されるルールを作成することができます。ネットワーク セキュリティ グループ ルールで、インターネットからの SSH トラフィックが許可されていることを確認します。詳細については、「[ネットワーク セキュリティ グループについて](../virtual-network/virtual-networks-nsg.md)」をご覧ください。

#### ソース 5: Linux ベースの Azure 仮想マシン

最後に考えられる問題のソースは、Azure 仮想マシン自体に関連するものです。

![Linux ベースの Azure 仮想マシンを示す図](./media/virtual-machines-linux-troubleshoot-ssh-connection/ssh-tshoot5.png)

[Linux 仮想マシンのパスワードまたは SSH をリセットする](virtual-machines-linux-classic-reset-access.md)手順をまだ実行していない場合は、仮想マシンでその手順に従います。

もう一度、コンピューターから接続を試みてください。まだ接続できない場合は、以下の問題が考えられます。

- SSH サービスがターゲット仮想マシンで実行されていない。
- SSH サービスが TCP ポート 22 をリッスンしていない。これをテストするには、ローカル コンピューターに telnet クライアントをインストールし、"telnet *cloudServiceName*.cloudapp.net 22" を実行します。これで、SSH エンドポイントに対する受信と送信の接続が仮想マシンで許可されているかどうかを確認できます。
- ターゲット仮想マシンのローカル ファイアウォールに、受信または送信の SSH トラフィックを妨げているルールがある。
- Azure 仮想マシンで実行されている侵入検出ソフトウェアまたはネットワーク監視ソフトウェアが、SSH 接続を妨げている。


## その他のリソース

[Linux 仮想マシンのパスワードまたは SSH をリセットする方法](virtual-machines-linux-classic-reset-access.md) (クラシック デプロイ モデルの仮想マシンの場合)

[Windows ベースの Azure 仮想マシンへの Windows リモート デスクトップ接続に関するトラブルシューティング](virtual-machines-windows-troubleshoot-rdp-connection.md)

[Azure 仮想マシンで実行されているアプリケーションへのアクセスに関するトラブルシューティング](virtual-machines-linux-troubleshoot-app-connection.md)

<!---HONumber=AcomDC_0406_2016-->