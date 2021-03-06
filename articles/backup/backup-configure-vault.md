---
title: "Windows Server を Azure にバックアップする (Resource Manager) | Microsoft Docs"
description: "バックアップ コンテナーの作成、資格情報のダウンロード、Backup エージェントのインストール、およびファイルとフォルダーの初回バックアップの完了によって、Windows サーバーまたはクライアントを Azure にバックアップします。"
services: backup
documentationcenter: 
author: markgalioto
manager: carmonm
editor: 
keywords: "バックアップ コンテナー; Windows サーバーのバックアップ; Windows のバックアップ;"
ms.assetid: 7f5b1943-b3c1-4ddb-8fb7-3560533c68d5
ms.service: backup
ms.workload: storage-backup-recovery
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 2/16/2017
ms.author: markgal;trinadhk;
translationtype: Human Translation
ms.sourcegitcommit: 1a87af9efeb6c00f3c67f2c2d8d8f2e0491d248d
ms.openlocfilehash: 018a1bde8163eda660fd50a41839b6c1ec622d79
ms.lasthandoff: 02/17/2017


---
# <a name="back-up-a-windows-server-or-client-to-azure-using-the-resource-manager-deployment-model"></a>Resource Manager デプロイメント モデルで Windows Server または Windows クライアントを Azure にバックアップする
> [!div class="op_single_selector"]
> * [Azure ポータル](backup-configure-vault.md)
> * [クラシック ポータル](backup-configure-vault-classic.md)
>
>

この記事では、Resource Manager デプロイメント モデルを使用して、Azure Backup で Windows Server または Windows クライアントのファイルやフォルダーを Azure にバックアップする方法について説明します。

[!INCLUDE [learn-about-deployment-models](../../includes/backup-deployment-models.md)]

![バックアップ プロセスの手順](./media/backup-configure-vault/initial-backup-process.png)

## <a name="before-you-start"></a>開始する前に
サーバー (またはクライアント) を Azure にバックアップするには、Azure アカウントが必要です。 アカウントがない場合は、 [無料アカウント](https://azure.microsoft.com/free/) を数分で作成できます。

## <a name="step-1-create-a-recovery-services-vault"></a>手順 1: Recovery Services コンテナーを作成する
Recovery Services コンテナーは、経時的に作成されたすべてのバックアップと回復ポイントを格納するエンティティです。 Recovery Services コンテナーには、保護対象のファイルとフォルダーに適用されるバックアップ ポリシーも含まれます。 Recovery Services コンテナーを作成するときは、適切なストレージ冗長オプションも選択することをお勧めします。

### <a name="to-create-a-recovery-services-vault"></a>Recovery Services コンテナーを作成するには
1. まだサインインしていない場合は、Azure サブスクリプションを使用して [Azure ポータル](https://portal.azure.com/) にサインインします。
2. ハブ メニューで **[参照]** をクリックし、リソースの一覧で「**Recovery Services**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Recovery Services コンテナー]**をクリックします。

    ![Create Recovery Services Vault step 1](./media/backup-configure-vault/browse-to-rs-vaults.png) <br/>

    Recovery Services コンテナーの一覧が表示されます。
3. **[Recovery Services コンテナー]** メニューの **[追加]** をクリックします。

    ![Create Recovery Services Vault step 2](./media/backup-configure-vault/rs-vault-menu.png)

    [Recovery Services コンテナー] ブレードが開き、**[名前]**、**[サブスクリプション]**、**[リソース グループ]**、**[場所]** を指定するよう求められます。

    ![Create Recovery Services vault step 5](./media/backup-configure-vault/rs-vault-attributes.png)
4. **[名前]**ボックスに、コンテナーを識別する表示名を入力します。 名前は Azure サブスクリプションに対して一意である必要があります。 2 ～ 50 文字の名前を入力します。 名前の先頭にはアルファベットを使用する必要があります。また、名前に使用できるのはアルファベット、数字、ハイフンのみです。
5. **[サブスクリプション]** をクリックして、使用可能なサブスクリプションの一覧を表示します。 どのサブスクリプションを使用すればよいかがわからない場合は、既定 (または推奨) のサブスクリプションを使用してください。 組織のアカウントが複数の Azure サブスクリプションに関連付けられている場合に限り、複数の選択肢が存在します。
6. **[リソース グループ]** をクリックして、使用可能なリソース グループを表示するか、**[新規]** をクリックして、新しいリソース グループを作成します。 リソース グループの詳細については、「[Azure Resource Manager の概要](../azure-resource-manager/resource-group-overview.md)」をご覧ください。
7. **[場所]** をクリックして、コンテナーの地理的リージョンを選択します。 この選択により、バックアップ データの送信先となるリージョンが決まります。 自分の場所から近いリージョンを選択することによって、Azure にバックアップする際のネットワーク待機時間を短縮できます。
8. **[作成]**をクリックします。 Recovery Services コンテナーの作成に時間がかかることがあります。 ポータルの右上隅で、状態の通知を監視します。 コンテナーが作成されると、ポータルで開かれます。 完了した後に、コンテナーが一覧に表示されない場合は、 **[最新の情報に更新]**をクリックします。 一覧が更新されたら、コンテナーの名前をクリックします。

### <a name="to-determine-storage-redundancy"></a>ストレージの冗長性を確認するには
初めて Recovery Services コンテナーを作成するときに、ストレージのレプリケーション方法を決定します。

1. コンテナーのダッシュボードと一緒に自動的に開く **[設定]** ブレードで、**[バックアップ インフラストラクチャ]** をクリックします。
2. [バックアップ インフラストラクチャ] ブレードで **[構成のバックアップ]** をクリックして、**[ストレージ レプリケーションの種類]** を表示します。

    ![Create Recovery Services vault step 5](./media/backup-configure-vault/backup-infrastructure.png)
3. コンテナーのストレージ レプリケーション オプションを選択します。

    ![List of recovery services vaults](./media/backup-configure-vault/choose-storage-configuration.png)

    既定では、コンテナーには geo 冗長ストレージがあります。 プライマリ バックアップ ストレージ エンドポイントとして Azure を使用している場合は、引き続き geo 冗長ストレージを使用します。 プライマリ以外のバックアップ ストレージ エンドポイントとして Azure を使用している場合は、ローカル冗長ストレージを選択します。ローカル冗長ストレージを選択すると、Azure にデータを格納するコストが削減されます。 [geo 冗長](../storage/storage-redundancy.md#geo-redundant-storage)ストレージ オプションと[ローカル冗長](../storage/storage-redundancy.md#locally-redundant-storage)ストレージ オプションの詳細については、こちらの[概要](../storage/storage-redundancy.md)を参照してください。

    コンテナーのストレージ オプションを選択したら、ファイルとフォルダーをコンテナーに関連付けることができます。

コンテナーを作成したら、Microsoft Azure Recovery Services エージェントをダウンロードしてインストールし、コンテナーの資格情報をダウンロードし、その資格情報を使用してエージェントをコンテナーに登録して、ファイルとフォルダーをバックアップするインフラストラクチャを準備します。

## <a name="step-2-download-files"></a>手順 2: ファイルをダウンロードする
> [!NOTE]
> 間もなく、Azure ポータルからバックアップを有効にできるようになります。 現時点では、オンプレミスの Microsoft Azure Recovery Services エージェントを使用して、ファイルやフォルダーをバックアップします。
>
>

1. Recovery Services コンテナーのダッシュボードで、 **[設定]** をクリックします。

    ![Open backup goal blade](./media/backup-configure-vault/settings-button.png)
2. [設定] ブレードで、**[作業の開始] > [Backup]** の順にクリックします。

    ![Open backup goal blade](./media/backup-configure-vault/getting-started-backup.png)
3. [Backup] ブレードで、 **[Backup goal]** をクリックします。

    ![Open backup goal blade](./media/backup-configure-vault/backup-goal.png)
4. [Where is your workload running (ワークロードの実行場所)] メニューの **[オンプレミス]** を選択します。
5. [What do you want to backup (バックアップ対象)] メニューの **[ファイルまたはフォルダー]** を選択し、**[OK]** をクリックします。

#### <a name="download-the-recovery-services-agent"></a>Recovery Services エージェントのダウンロード
1. **[インフラストラクチャの準備]** ブレードで、**[Windows Server または Windows クライアント用エージェントのダウンロード]** をクリックします。

    ![[Download Agent for Windows Server or Windows Client]](./media/backup-configure-vault/prepare-infrastructure-short.png)
2. ダウンロードのポップアップ ウィンドウで、 **[保存]** をクリックします。 既定では、 **MARSagentinstaller.exe** ファイルがダウンロード フォルダーに保存されます。

#### <a name="download-vault-credentials"></a>コンテナー資格情報のダウンロード
1. [インフラストラクチャの準備] ブレードで、**[ダウンロード]、[保存]** の順にクリックします。

    ![[Download Agent for Windows Server or Windows Client]](./media/backup-configure-vault/prepare-infrastructure-download.png)

## <a name="step-3-install-and-register-the-agent"></a>手順 3: エージェントをインストールして登録する
1. ダウンロード フォルダー (または他の保存場所) で **MARSagentinstaller.exe** を見つけて、ダブルクリックします。
2. Microsoft Azure Recovery Services エージェント セットアップ ウィザードを実行します。 ウィザードでは以下のことを行う必要があります。

   * インストールとキャッシュ フォルダーの場所を選択します。
   * プロキシ サーバーを使用してインターネットに接続する場合は、プロキシ サーバーの情報を指定します。
   * 認証済みのプロキシを使用する場合は、ユーザー名とパスワードの詳細を入力します。
   * ダウンロードしたコンテナーの資格情報を指定します。
   * 暗号化パスフレーズを安全な場所に保存します。

     > [!NOTE]
     > パスフレーズを紛失または忘れた場合、Microsoft はバックアップ データの回復を支援することはできません。 安全な場所にファイルを保存してください。 バックアップを復元するために必要です。
     >
     >

エージェントがインストールされ、コンピューターがコンテナーに登録されました。 バックアップを構成してスケジュールする準備ができました。

### <a name="confirm-the-installation"></a>インストールの構成
エージェントが適切にインストールおよび登録されていることを確認するには、管理ポータルの **[実稼働サーバー]** セクションでバックアップした項目をチェックします。 これを行うには、次の手順を実行します。

1. Azure サブスクリプションを使用して、 [Azure ポータル](https://portal.azure.com/) にサインインします。
2. ハブ メニューで **[参照]** をクリックし、リソースの一覧で「**Recovery Services**」と入力します。 入力を始めると、入力内容に基づいて、一覧がフィルター処理されます。 **[Recovery Services コンテナー]**をクリックします。

    ![Create Recovery Services Vault step 1](./media/backup-configure-vault/browse-to-rs-vaults.png) <br/>

    Recovery Services コンテナーの一覧が表示されます。
3. 作成したコンテナーの名前を選択します。

    Recovery Services コンテナーのダッシュボード ブレードが開きます。

    ![Recovery Services コンテナーのダッシュボード](./media/backup-configure-vault/rs-vault-dashboard.png) <br/>
4. ページの上部にある **[設定]** ボタンをクリックします。
5. **[バックアップ インフラストラクチャ]、[実稼働サーバー]** の順にクリックします。

    ![実稼働サーバー](./media/backup-configure-vault/production-server-verification.png)

サーバーが一覧に示されている場合は、エージェントは正しくインストールおよび登録されています。

## <a name="step-4-complete-the-initial-backup"></a>手順 4: 初回バックアップを完了する
初回バックアップには、次の&2; つの主要なタスクが含まれています。

* バックアップのスケジュール
* 初回のファイルとフォルダーのバックアップ

初回バックアップを実行するには、Microsoft Azure Backup エージェントを使用します。

### <a name="to-schedule-the-backup"></a>バックアップのスケジュールを設定するには
1. Microsoft Azure Backup エージェントを開きます  エージェントは、コンピューターで **Microsoft Azure Backup**を検索すると見つかります。

    ![Launch the Azure Backup agent](./media/backup-configure-vault/snap-in-search.png)
2. Backup エージェントで、 **[バックアップのスケジュール]**をクリックします。

    ![Schedule a Windows Server backup](./media/backup-configure-vault/schedule-first-backup.png)
3. バックアップのスケジュール ウィザードの [作業の開始] ページで、 **[次へ]**をクリックします。
4. [バックアップする項目の選択] 画面で、 **[項目の追加]**をクリックします。
5. バックアップするファイルとフォルダーを選択し、 **[OK]**をクリックします。
6. ページの下部にある **[次へ]**」を参照してください。
7. **[バックアップ スケジュールの選択]** ページで**バックアップ スケジュール**を指定し、**[次へ]** をクリックします。

    毎日 (1 日に最大&3; 回) または毎週のバックアップをスケジュールすることができます。

    ![Windows Server のバックアップ項目](./media/backup-configure-vault/specify-backup-schedule-close.png)

   > [!NOTE]
   > バックアップ スケジュールを指定する方法の詳細については、「 [Azure Backup を使用してテープのインフラストラクチャを置換する](backup-azure-backup-cloud-as-tape.md)」を参照してください。
   >
   >
8. **[保持ポリシーの選択]** ページで、バックアップ コピーの**リテンション期間ポリシー**を選択します。

    保有ポリシーは、バックアップを格納する必要がある期間を指定します。 すべてのバックアップ ポイントに "同じポリシー" を指定するのでなく、バックアップが実行されるタイミングに基づいて異なる保持ポリシーを指定できます。 必要に応じて、日、週、月、および年単位で保有ポリシーを変更できます。
9. [初期バックアップの種類の選択] ページで、初期バックアップの種類を選択します。 **[自動でネットワーク経由]** オプションが選択された状態のままにし、**[次へ]** をクリックします。

    ネットワーク経由で自動的にバックアップことも、オフラインでバックアップすることもできます。 この記事の残りの部分では、自動バックアップのプロセスについて説明します。 オフライン バックアップを実行する場合は、「 [Azure Backup でのオフライン バックアップのワークフロー](backup-azure-backup-import-export.md) 」で詳細を確認してください。
10. [確認] ページで情報を確認し、 **[完了]**をクリックします。
11. ウィザードでバックアップ スケジュールの作成が完了したら、 **[閉じる]**をクリックします。

### <a name="enable-network-throttling"></a>ネットワーク調整の有効化
Backup エージェントは、ネットワーク スロットルを提供します。 調整では、データ転送中にネットワーク帯域幅をどのように使用するかを制御します。 データのバックアップを業務時間中に行う必要があり、バックアップ処理が他のインターネット トラフィックに影響を与えないようにする場合などに、この制御は便利です。 調整はバックアップと復元のアクティビティに適用されます。

> [!NOTE]
> ネットワーク スロットルは、Windows Server 2008 R2 SP1、Windows Server 2008 SP2、Windows 7 (Service Pack を含む) では使用できません。 Azure Backup のネットワーク スロットル機能は、ローカル オペレーティング システムのサービス品質 (QoS) を利用します。 Azure Backup で上記のオペレーティング システムを保護することはできますが、これらのプラットフォームで使用できる QoS のバージョンでは、Azure Backup のネットワーク スロットルが機能しません。 ネットワーク スロットルは、上記以外の [サポートされているすべてのオペレーティング システム](backup-azure-backup-faq.md)で使用できます。
>
>

**ネットワーク調整を有効にするには**

1. Backup エージェントで、 **[プロパティの変更]**をクリックします。

    ![[プロパティの変更]](./media/backup-configure-vault/change-properties.png)
2. **[調整]** タブで、**[バックアップ操作用のインターネット使用帯域幅の調整を有効にする]** チェック ボックスをオンにします。

    ![ネットワークのスロットル](./media/backup-configure-vault/throttling-dialog.png)
3. スロットルを有効にした後、**[作業時間]** と **[作業時間外]** で、バックアップ データ転送のために許可される帯域幅を指定します。

    帯域幅の値は、512 キロバイト/秒 (Kbps) から始まり、最大で 1023 メガバイト/秒 (Mbps) まで指定できます。 また、 **[作業時間]**の開始および終了時刻や、作業日と見なされる曜日も指定できます。 指定した作業時間以外の時間は、作業時間外と見なされます。
4. **[OK]**をクリックします。

### <a name="to-back-up-files-and-folders-for-the-first-time"></a>初回のファイルとフォルダーをバックアップするには
1. Backup エージェントで **[今すぐバックアップ]** をクリックして、ネットワーク経由での最初のシード処理を完了します。

    ![Windows Server を今すぐバックアップする](./media/backup-configure-vault/backup-now.png)
2. [確認] ページで、今すぐバックアップ ウィザードによってコンピューターのバックアップに使用される設定を確認します。 次に、 **[バックアップ]**をクリックします。
3. **[閉じる]** をクリックしてウィザードを閉じます。 バックアップ プロセスが完了する前にウィザードを閉じても、ウィザードはバックグラウンドで引き続き実行されます。

初回バックアップが完了すると、 **[ジョブは完了しました]** 状態が Backup コンソールに表示されます。

![IR complete](./media/backup-configure-vault/ircomplete.png)

## <a name="questions"></a>疑問がある場合
ご不明な点がある場合や今後搭載を希望する機能がある場合は、 [フィードバックをお送りください](http://aka.ms/azurebackup_feedback)。

## <a name="next-steps"></a>次のステップ
VM や他のワークロードのバックアップの詳細については、以下を参照してください。

* ファイルとフォルダーをバックアップしたので、 [コンテナーとサーバーを管理](backup-azure-manage-windows-server.md)できます。
* バックアップを復元する必要がある場合は、 [Windows コンピューターへのファイルの復元](backup-azure-restore-windows-server.md)に関する記事を参照してください。

