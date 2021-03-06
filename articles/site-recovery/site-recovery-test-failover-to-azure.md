---
title: "Site Recovery での Azure へのフェールオーバーをテストする | Microsoft Docs"
description: "オンプレミスから Azure へのテスト フェールオーバーの実行について説明します"
services: site-recovery
documentationcenter: 
author: prateek9us
manager: gauravd
editor: 
ms.assetid: 44813a48-c680-4581-a92e-cecc57cc3b1e
ms.service: site-recovery
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: storage-backup-recovery
ms.date: 2/15/2017
ms.author: pratshar
translationtype: Human Translation
ms.sourcegitcommit: 753d70b9b9a4c2805033f2f2fb293dbdef7b4db4
ms.openlocfilehash: 27ee6bbadd2b03ccf60739a276f49783ea68f9d1
ms.lasthandoff: 02/15/2017


---
# <a name="test-----failover-to-azure-in-site-recovery"></a>Site Recovery での Azure へのフェールオーバーをテストする

この記事では、復旧サイトとして Azure を使う Site Recovery で保護されている仮想マシンと物理サーバーのテスト フェールオーバーまたは DR ドリルを行うための情報と手順について説明します。

コメントや質問はこの記事の末尾、または [Azure Recovery Services フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=hypervrecovmgr)で投稿してください。

テスト フェールオーバーは、データの損失またはダウンタイムを発生させることなく、レプリケーション戦略の検証または障害復旧の訓練を行うために実行します。 テスト フェールオーバーを行うことで、実行中のレプリケーションまたは運用環境に影響が及ぶことはありません。 テスト フェールオーバーは、仮想マシンまたは[復旧計画](site-recovery-create-recovery-plans.md)で行うことができます。 テスト フェールオーバーをトリガーするときは、テスト仮想マシンの接続先のネットワークを指定する必要があります。 テスト フェールオーバーをトリガーした後は、**[ジョブ]** ページで進行状況を追跡できます。  


## <a name="supported-scenarios"></a>サポートされるシナリオ
テスト フェールオーバーは、[レガシ VMware サイトの Azure へのレプリケート](site-recovery-vmware-to-azure-classic-legacy.md)を除くすべてのデプロイ シナリオでサポートされます。 テスト フェールオーバーは、Azure に仮想マシンがフェールオーバーされたときもサポートされません。  


## <a name="run-a-test-failover"></a>テスト フェールオーバーの実行
この手順では、復旧計画のテスト フェールオーバーを実行する方法について説明します。 または、適切なオプションを使うことで、単一マシンに対するテスト フェールオーバーを実行することもできます。

1. **[復旧計画]**  >  *recoveryplan_name* を選択します。 **[テスト フェールオーバー]**クリックします。

    ![[テスト フェールオーバー]](./media/site-recovery-test-failover-to-azure/test-failover1.png)
1. フェールオーバーする**復旧ポイント**を選びます。 次のいずれかのオプションを使うことができます。
    1.  **Latest processed (最新の処理済み)**: このオプションは、復旧計画のすべての仮想マシンを、Site Recovery サービスによって既に処理された最新の復旧ポイントにフェールオーバーします。 仮想マシンのテスト フェールオーバーを行っているときは、最新の処理済み復旧ポイントのタイムスタンプも表示されます。 復旧計画のフェールオーバーを行っている場合は、個別の仮想マシンに移動し、**[最新の復旧ポイント]** タイルを見てこの情報を取得できます。 未処理データの処理に時間は費やされないので、このオプションは低 RTO (目標復旧時間) のフェールオーバー オプションを提供します。
    1.    **Latest app-consistent (最新のアプリ整合性)**: このオプションは、復旧計画のすべての仮想マシンを、Site Recovery サービスによって既に処理された最新のアプリケーション整合性復旧ポイントにフェールオーバーします。 仮想マシンのテスト フェールオーバーを行っているときは、最新のアプリ整合性復旧ポイントのタイムスタンプも表示されます。 復旧計画のフェールオーバーを行っている場合は、個別の仮想マシンに移動し、**[最新の復旧ポイント]** タイルを見てこの情報を取得できます。
    1.    **Latest (最新)**: このオプションは、Site Recovery サービスに送信されたすべてのデータを最初に処理して、各仮想マシンの復旧ポイントを作成してから、それにフェールオーバーします。 このオプションは、フェールオーバー後に作成された仮想マシンが、フェールオーバーがトリガーされた時点で Site Recovery サービスにレプリケートされたすべてのデータを含むようになるので、最も低い RPO (目標復旧時点) を提供します。
    1.    **Custom (カスタム)**: 仮想マシンのテスト フェールオーバーを行う場合は、このオプションを使って特定の復旧ポイントにフェールオーバーできます。
1. **Azure 仮想ネットワーク**を選びます。テスト用の仮想マシンが作成される Azure 仮想ネットワークを指定します。 Site Recovery は、仮想マシンの **[コンピューティングとネットワーク]** の設定で提供されたものと同じ名前のサブネットに、同じ IP を使って、テスト仮想マシンの作成を試みます。 テスト フェールオーバー用に提供された Azure 仮想ネットワークで同じ名前のサブネットを使用できない場合は、アルファベット順で最初のサブネットにテスト仮想マシンが作成されます。 同じ IP をサブネットで使うことができない場合、仮想マシンはサブネットで利用できる別の IP アドレスを受け取ります。 [詳しく](#creating-a-network-for-test-failover)は、このセクションをご覧ください。
1. Azure にフェールオーバーしていて、データ暗号化が有効になっている場合は、**[暗号化キー]** で、プロバイダーのインストール中にデータ暗号化を有効にしたときに発行された証明書を選びます。 仮想マシンで暗号化を有効にしていない場合は、この手順を省略できます。
1. **[ジョブ]** タブで、フェールオーバーの進行状況を追跡します。 テスト レプリカ マシンも Azure ポータルで確認できます。
1. 仮想マシンで RDP 接続を開始するには、フェールオーバーされる仮想マシンのネットワーク インターフェイスで、[パブリック IP を追加](site-recovery-monitoring-and-troubleshooting.md#adding-a-public-ip-on-a-resource-manager-virtual-machine)する必要があります。 クラシック仮想マシンにフェールオーバーする場合は、ポート 3389 で[エンドポイントを追加](../virtual-machines/virtual-machines-windows-classic-setup-endpoints.md)する必要があります。
1. 完了したら、復旧計画の **[Cleanup test failover (テスト フェールオーバーのクリーンアップ)]** をクリックします。 **[メモ]** を使用して、テスト フェールオーバーに関連する観察結果をすべて記録し、保存します。 これで、テスト フェールオーバー中に作成された仮想マシンが削除されます。


> [!TIP]
> Site Recovery は、仮想マシンの **[コンピューティングとネットワーク]** の設定で提供されたものと同じ名前のサブネットに、同じ IP を使って、テスト仮想マシンの作成を試みます。 テスト フェールオーバー用に提供された Azure 仮想ネットワークで同じ名前のサブネットを使用できない場合は、アルファベット順で最初のサブネットにテスト仮想マシンが作成されます。 同じ IP をサブネットで使うことができない場合、仮想マシンはサブネットで利用できる別の IP アドレスを受け取ります。 
>
>


## <a name="creating-a-network-for-test-failover"></a>テスト フェールオーバー用のネットワークを作成する
テスト フェールオーバーを行うときは、仮想マシンの **[コンピューティングとネットワーク]** の設定で指定した運用復旧サイト ネットワークから分離されたネットワークを選ぶことをお勧めします。 既定では、作成した Azure 仮想ネットワークは、他のネットワークから分離されます。 このネットワークは、運用ネットワークと同じものにする必要があります。

1. テスト ネットワークは、運用ネットワークと同じ数および同じ名前のサブネットを持つ必要があります。
1. テスト ネットワークは、運用ネットワークと同じ IP 範囲を使う必要があります。
1. **[コンピューティングとネットワーク]** の設定で DNS 仮想マシンのターゲット IP として指定した IP に、テスト用ネットワークの DNS を更新します。 詳細については、 [Active Directory 用のテスト フェールオーバーの考慮事項](site-recovery-active-directory.md#test-failover-considerations) を参照してください。


## <a name="test-failover-to-a-production-network-on-recovery-site"></a>復旧サイトの運用ネットワークへのテスト フェールオーバー
テスト フェールオーバーを行うときは、仮想マシンの **[コンピューティングとネットワーク]** の設定で指定した運用復旧サイト ネットワークとは異なるネットワークを選ぶことをお勧めします。 ただし、フェールオーバーされた仮想マシンでエンド ツー エンドのネットワーク接続を実際に検証したい場合は、次の点に注意してください。

1. テスト フェールオーバーを実行するときは、プライマリ仮想マシンがシャットダウンされていることを確認します。 そうしないと、同じ ID を持つ&2; つの仮想マシンが同じネットワークで同時に実行することになり、望ましくない結果を生じる可能性があります。
1. テスト フェールオーバー仮想マシンに加えた変更は、テスト フェールオーバー仮想マシンをクリーンアップすると失われます。 これらの変更は、プライマリ仮想マシンにはレプリケートされません。
1. このようなテスト方法では、運用アプリケーションのダウンタイムにつながります。 DR ドリルの実行中はアプリケーションのユーザーにアプリケーションを使わないよう指示する必要があります。  



## <a name="prepare-active-directory-and-dns"></a>Active Directory と DNS の準備
アプリケーションのテストのためにテスト フェールオーバーを実行するには、テスト環境内に Active Directory 運用環境のコピーが必要です。 詳細については、 [Active Directory 用のテスト フェールオーバーの考慮事項](site-recovery-active-directory.md#test-failover-considerations) を参照してください。

## <a name="prepare-to-connect-to-azure-vms-after-failover"></a>フェールオーバー後に Azure VM に接続するための準備をする

フェールオーバー後に RDP を使用して Azure VM に接続する場合は、次の表に示されているアクションを実行する必要があります。

**フェールオーバー** | **場所** | **アクション**
--- | --- | ---
**Windows で実行中の Azure VM** | フェールオーバー前にオンプレミスのコンピューターで | インターネット経由で Azure VM にアクセスするには、RDP を有効にし、TCP と UDP の規則が **[パブリック]** に追加されていることを確認し、**[Windows ファイアウォール]** > **[許可されたアプリ]** のすべてのプロファイルで RDP が許可されていることを確認します。<br/><br/> サイト間接続でアクセスするには、コンピューターで RDP を有効にし、**[Windows ファイアウォール]** -> **[許可されたアプリおよび機能]** の **[ドメイン] と [プライベート]** ネットワークで RDP が許可されていることを確認します。<br/><br/> Azure VM エージェントのインストール<br/><br/> オペレーティング システムの SAN ポリシーが **[OnlineAll]** に設定されていることを確認します。 [詳細情報](https://support.microsoft.com/kb/3031135)。<br/><br/> フェールオーバーを実行する前に、IPSec サービスを無効にします。
**Windows で実行中の Azure VM** | フェールオーバー後に Azure VM で | RDP プロトコル (ポート 3389) のパブリック エンドポイントを追加し、ログインの資格情報を指定します。<br/><br/> パブリック アドレスを使用する仮想マシンへの接続を妨げるドメイン ポリシーを使用していないことを確認します。<br/><br/> 接続を試みます。 接続できない場合は、VM が実行されていることを確認します。 [トラブルシューティングのヒント](http://social.technet.microsoft.com/wiki/contents/articles/31666.troubleshooting-remote-desktop-connection-after-failover-using-asr.aspx)を取得します。<br/><br/> オペレーティング システムの SAN ポリシーが **[OnlineAll]** に設定されていることを確認します。 [詳細情報](https://support.microsoft.com/kb/3031135)。<br/><br/> フェールオーバーを実行する前に、IPSec サービスを無効にします。<br/><br/> コンピューターまたはサブネットに関連付けられているネットワーク セキュリティ グループがある場合、そのグループに HTTP/HTTPS を許可する送信規則が設定されていることを確認します。 VM のフェールオーバー先となるネットワークの DNS が正しく構成されていることを確認します。 これらが行われていない場合、"PreFailoverWorkflow タスクの WaitForScriptExecutionTask がタイムアウトしました" というエラーでタイムアウトになる可能性があります。 [詳細情報](https://github.com/rayne-wiselman/azure-docs-pr/blob/75653b84d6ccbefe7d5230449bea81f498e10a98/articles/site-recovery/site-recovery-monitoring-and-troubleshooting.md#recovery)。
**Linux で実行中の Azure VM** | フェールオーバー前にオンプレミスのコンピューターで | Azure VM 上の Secure Shell サービスがシステム起動時に自動的に開始されるよう設定されていることを確認します。<br/><br/> ファイアウォール規則で SSH 接続が許可されていることを確認します。
**Linux で実行中の Azure VM** | フェールオーバー後に Azure VM で | フェールオーバーされた VM と接続先の Azure サブネットのネットワーク セキュリティ グループの規則で、SSH ポートへの着信接続を許可する必要があります。<br/><br/> SSH ポート (既定では TCP ポート 22) での受信接続を許可するために、パブリック エンドポイントを作成する必要があります。<br/><br/> VM が VPN 接続 (ExpressRoute またはサイト間 VPN) 経由でアクセスされる場合は、クライアントを使用して SSH 経由で直接 VM に接続できます。<br/><br/> コンピューターまたはサブネットに関連付けられているネットワーク セキュリティ グループがある場合、そのグループに HTTP/HTTPS を許可する送信規則が設定されていることを確認します。 VM のフェールオーバー先となるネットワークの DNS が正しく構成されていることを確認します。 これらが行われていない場合、"PreFailoverWorkflow タスクの WaitForScriptExecutionTask がタイムアウトしました" というエラーでタイムアウトになる可能性があります。 [詳細情報](https://github.com/rayne-wiselman/azure-docs-pr/blob/75653b84d6ccbefe7d5230449bea81f498e10a98/articles/site-recovery/site-recovery-monitoring-and-troubleshooting.md#recovery)。



## <a name="next-steps"></a>次のステップ
テスト フェールオーバーが正常に行われた後は、[フェールオーバー](site-recovery-failover.md)を試すことができます。

