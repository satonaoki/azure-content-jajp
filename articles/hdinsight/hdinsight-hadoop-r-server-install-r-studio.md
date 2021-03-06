<properties
	pageTitle="R Server がインストールされた HDInsight クラスターに RStudio をインストールする | Microsoft Azure"
	description="HDInsight クラスターの R Server に RStudio をインストールする方法。"
	services="hdinsight"
	documentationCenter=""
	authors="jeffstokes72"
	manager="paulettm"
	editor="cgronlun"/>

<tags
   ms.service="hdinsight"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="big-data"
   ms.date="03/29/2016"
   ms.author="jeffstok"/>


# R Server を使用して RStudio を HDInsight クラスターにインストールする

現在、R に使用できる統合開発環境 (IDE) は複数あります。たとえば、Microsoft から最近発表された [R Tools for Visual Studio](https://www.visualstudio.com/ja-JP/features/rtvs-vs.aspx) (RTVS) は、[RStudio](https://www.rstudio.com/products/rstudio-server/) のデスクトップおよびサーバー ツールのファミリです。また、Walware の Eclipse ベースの [StatET](http://www.walware.de/goto/statet) などがあります。Linux で最もよく使用されているのは、[RStudio Server](https://www.rstudio.com/products/rstudio-server/) です。RStudio Server には、リモート クライアントに使用するブラウザーベースの IDE があります。HDInsight Premium クラスターのエッジ ノードに RStudio Server をインストールすると、クラスターの R Server を使用した R スクリプトの開発と実行に IDE の機能をフルに活用できるようになり、既定の R コンソールを使用した場合よりも生産性が大幅に向上します。

この記事では、カスタム スクリプトを使用して、クラスターのエッジ ノードにコミュニティ (無料) バージョンの RStudio Server をインストールする方法について説明します。商用ライセンスが提供される Pro バージョンの RStudio Server の方がよい場合は、[RStudio Server](https://www.rstudio.com/products/rstudio/download-server/) のインストール手順に従って操作してください。

## 前提条件

* R Server がインストールされた Azure HDInsight クラスター。手順については、[HDInsight クラスター上の R Server の概要](hdinsight-hadoop-r-server-get-started.mdulet)のページを参照してください。
* SSH クライアント。Linux および UNIX のディストリビューション、または Macintosh OS X の場合、オペレーティング システムに `ssh`コマンドが用意されています。Windows の場合は [PuTTY](http://www.chiark.greenend.org.uk/~sgtatham/putty/download.html) をお勧めします。 


## カスタム スクリプトを使用してクラスターに RStudio をインストールする

1. クラスターのエッジ ノードを特定します。R Server がインストールされた HDInsight クラスターの場合、ヘッド ノードとエッジ ノードには次の命名規則があります。

	* ヘッド ノード - `CLUSTERNAME-ssh.azurehdinsight.net`
	* エッジ ノード - `rserver.CLUSTERNAME.ssh.azurehdinsight.net` 

3. 上記の命名パターンを使用して、クラスターのエッジ ノードに SSH でログインします。
 
	* Linux クライアントから接続する場合は、「[Linux ベースの HDInsight クラスターへの接続](hdinsight-hadoop-linux-use-ssh-unix.md#connect-to-a-linux-based-hdinsight-cluster)」を参照してください。
	* Windows クライアントから接続する場合は、「[HDInsight の Linux ベースの Hadoop で Windows から SSH を使用する](hdinsight-hadoop-linux-use-ssh-windows.md#connect-to-a-linux-based-hdinsight-cluster)」を参照してください。

2. 接続したら、クラスターのルート ユーザーになります。SSH セッションでは、次のコマンドを使用します。

		sudo su -

3. カスタム スクリプトをダウンロードして RStudio をインストールします。次のコマンドを使用します。

		wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/InstallRStudio.sh

4. カスタム スクリプト ファイルのアクセス許可を変更し、スクリプトを実行します。次のコマンドを使用します。

		chmod 755 InstallRStudio.sh
		./InstallRStudio.sh

5. R Server がインストールされた HDInsight クラスターを作成するときに SSH パスワードを使用した場合は、この手順をスキップし、次に進むことができます。クラスターを作成する代わりに SSH キーを使用した場合、SSH ユーザーのパスワードを設定する必要があります。RStudio に接続するときに、このパスワードが必要になります。次のコマンドを実行します。**現在の Kerberos パスワード**の入力を求められたら、そのまま **Enter** キーを押します。

		passwd remoteuser
		Current Kerberos password:
		New password:
		Retype new password:
		Current Kerberos password:
		
	パスワードを正しく設定すると、次のようなメッセージが表示されます。

		passwd: password updated successfully


	SSH セッションを終了します。

6. クラスターへの SSH トンネルを作成するには、HDInsight クラスターの `localhost:8787` からクライアント コンピューターにマッピングします。SSH トンネルを作成してから、新しいブラウザー セッションを開く必要があります。

	* ([Cygwin](http://www.redhat.com/services/custom/cygwin/) を使用する) Linux クライアントまたは Windows クライアントで、ターミナル セッションを開き、次のコマンドを実行します。

			ssh -L localhost:8787:localhost:8787 USERNAME@rserver.CLUSTERNAME.ssh.azurehdinsight.net
			
		**USERNAME** は、実際の HDInsight クラスターの SSH ユーザーに置き換えます。また、**CLUSTERNAME** は、HDInsight クラスターの名前に置き換えます。

	* Windows クライアントで、SSH トンネル用の PuTTY を作成します。

		1.  PuTTY を開き、接続情報を入力します。HDInsight で PuTTY を使用する方法については、「[HDInsight の Linux ベースの Hadoop で Windows から SSH を使用する](hdinsight-hadoop-linux-use-ssh-windows.md)」を参照してください。
		2.  ダイアログの左にある **[Category]** セクションで、**[Connection]**、**[SSH]** の順に展開し、**[Tunnels]** を選択します。
		3.  **[Options controlling SSH port forwarding]** フォームに次の情報を入力します。

			* **[Source port]** - 転送するクライアント上のポート。たとえば、「**8787**」と入力します。
			* **[Destination]** - ローカル クライアント コンピューターにマッピングする宛先。たとえば、「**localhost:8787**」と入力します。

			![SSH トンネルを作成する](./media/hdinsight-hadoop-r-server-install-r-studio/createsshtunnel.png "SSH トンネルを作成する")

		4. **[Add]** をクリックして設定を追加し、**[Open]** をクリックして SSH 接続を開きます。
		5. プロンプトが表示されたら、サーバーにログインします。これにより、SSH セッションが確立され、トンネルが有効になります。

7. Web ブラウザーを開き、トンネルに入力したポートに基づいて次の URL を入力します。

		http://localhost:8787/ 

8. SSH のユーザー名とパスワードを入力してクラスターに接続するように求められます。クラスターの作成時に SSH キーを使用した場合、前述の手順 5 で作成したパスワードを入力する必要があります。

	![R Studio に接続する](./media/hdinsight-hadoop-r-server-install-r-studio/connecttostudio.png "SSH トンネルを作成する")

9. RStudio のインストールが成功したかどうかをテストするには、R ベースの MapReduce および Spark ジョブをクラスターで実行するテスト スクリプトを実行します。SSH コンソールに戻り、次のコマンドを入力して、テスト スクリプトをダウンロードし、RStudio で実行します。

	* R で Hadoop クラスターを作成した場合は、次のコマンドを実行します。
		
			wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi.r

	* R で Spark クラスターを作成した場合は、次のコマンドを実行します。

			wget http://mrsactionscripts.blob.core.windows.net/rstudio-server-community-v01/testhdi_spark.r

10. RStudio に、ダウンロードしたテスト スクリプトが表示されます。ファイルをダブルクリックして開き、ファイルの内容を選択して **[Run]** をクリックします。**[Console]** ウィンドウに出力が表示されます。
 
	![インストールをテストする](./media/hdinsight-hadoop-r-server-install-r-studio/test-r-script.png "インストールをテストする")

## 関連項目

- [Compute context options for R Server on HDInsight clusters (HDInsight クラスターでの R Server のコンピューティング コンテキストのオプション)](hdinsight-hadoop-r-server-compute-contexts.md)

- [Azure Storage options for R Server on HDInsight Premium (HDInsight Premium での R Server の Azure Storage オプション)](hdinsight-hadoop-r-server-storage.md)


 

<!---HONumber=AcomDC_0330_2016------>