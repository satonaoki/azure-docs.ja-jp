---
title: "Azure IaaS VM ディスクについてよく寄せられる質問 (FAQ) | Microsoft Docs"
description: "Azure IaaS VM ディスクと Premium ディスク (管理および非管理) についてよく寄せられる質問"
services: storage
documentationcenter: 
author: ramankumarlive
manager: aungoo-msft
editor: tysonn
ms.assetid: e2a20625-6224-4187-8401-abadc8f1de91
ms.service: storage
ms.workload: storage
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 02/06/2017
ms.author: ramankum
translationtype: Human Translation
ms.sourcegitcommit: 0746c954e669bd739b8ecfcddaf287cb5172877f
ms.openlocfilehash: 95b627738726f3c108fff38bfeda413303b2c718


---
# <a name="frequently-asked-questions-about-azure-iaas-vm-disks-and-managed-and-unmanaged-premium-disks"></a>Azure IaaS VM ディスクと Premium 管理ディスクおよび非管理ディスクについてよく寄せられる質問

この記事では、Managed Disks と Premium Storage についてよく寄せられる質問をいくつか取り上げます。

## <a name="managed-disks"></a>Managed Disks

**Azure Managed Disks とは何ですか?**

Managed Disks は、ユーザーに代わってストレージ アカウントの管理を行うことで Azure IaaS VM のディスク管理を簡略化する機能です。 詳細については、「[Managed Disks Overview](storage-managed-disks-overview.md)」(Managed Disks の概要) を参照してください。

**Standard 管理ディスクをサイズが 80 GB の既存の VHD から作成した場合、どのくらいの料金がかかりますか?**

80 GB の VHD から作成した Standard 管理ディスクは、1 つ上の Standard ディスク サイズすなわち S10 ディスクとして扱われます。 従って、S10 ディスクの料金が課金されます。 詳細については[価格のページ](https://azure.microsoft.com/pricing/details/storage)をご覧ください。

**Standard 管理ディスクのトランザクション コストはありますか?**

はい、トランザクションごとに課金されます。 詳細については価格のページ (https://azure.microsoft.com/pricing/details/storage) をご覧ください。

**Standard 管理ディスクでは、ディスク上の実際のデータ サイズについて課金されますか? または、ディスクのプロビジョニング容量について課金されますか?**

ディスクのプロビジョニング容量に基づいて課金されます。 詳細については[価格のページ](https://azure.microsoft.com/pricing/details/storage)をご覧ください。

**Premium 管理ディスクと非管理ディスクの価格設定はどのように異なりますか?**

Premium 管理ディスクの価格は Premium 非管理ディスクと同じです。

**管理ディスクのストレージ アカウントのタイプ (Standard/Premium) を変更できますか?**

はい。 管理ディスクのストレージ アカウント タイプは、Azure Portal、PowerShell、または Azure CLI を使用して変更できます。

**管理ディスクをプライベート ストレージ アカウントにコピーまたはエクスポートする方法はありますか?**

はい。Azure Portal、PowerShell、または Azure CLI を使用して管理ディスクをエクスポートできます。

**Azure ストレージ アカウントの VHD ファイルを使用して、別のサブスクリプションに管理ディスクを作成できますか?**

いいえ。

**Azure ストレージ アカウントの VHD ファイルを使用して、別のリージョンに管理ディスクを作成できますか?**

いいえ。

**管理ディスクを使用するユーザーが容量を制限されることはありますか?**

Managed Disks では、ストレージ アカウントに関連する制限が排除されています。 ただし、サブスクリプションごとの管理ディスク数は既定では 2000 個に制限されます。 サポートに連絡して制限を増やすことができます。

**管理ディスクの増分スナップショットを作成できますか?**

いいえ。 現在のスナップショット機能では、管理ディスクの完全なコピーが作成されます。 ただし、将来的に増分スナップショットをサポートする予定があります。

**可用性セットの VM で管理ディスクと非管理ディスクを混在させることができますか?**

いいえ、可用性セット内の VM は、すべて管理ディスクまたはすべて非管理ディスクを使用する必要があります。 可用性セットを作成するときに、使用するディスクの種類を選択できます。

**Managed Disks は Azure Portal の既定オプションですか?**

現在は違いますが、将来的には既定になります。

**空の管理ディスクを作成できますか?**

はい、空のディスクを作成できます。 管理ディスクは VM とは独立して作成できます。つまり VM に接続せずに作成できます。

**Managed Disks を使用する可用性セットでサポートされる障害ドメイン数はいくつですか?**

Managed Disks を使用する可用性セットでサポートされる障害ドメイン数は 2 または 3 です。これは、配置されているリージョンによって異なります。

**診断のための Standard ストレージ アカウントはどのように設定されますか?**

VM 診断用にプライベート ストレージ アカウントを設定します。 今後は、診断も Managed Disks に切り替える予定です。

**Managed Disks ではどのような RBAC サポートがありますか?**

Managed Disks では&3; つの重要な既定のロールがサポートされます。

1.  所有者: アクセス権を含めすべてを管理できます。

2.  共同作業者: アクセス権以外のすべてを管理できます。

3.  閲覧者: すべてを閲覧できますが、変更することはできません。

**管理ディスクをプライベート ストレージ アカウントにコピーまたはエクスポートする方法はありますか?**

管理ディスクの読み取り専用 Shared Access Signature (SAS) URI を取得でき、それを使用して、内容をプライベート ストレージ アカウントまたはオンプレミス ストレージにコピーできます。

**管理ディスクのコピーを作成できますか?**

管理ディスクのスナップショットを作成し、そのスナップショットを使用して別の管理ディスクを作成できます。

**非管理ディスクはまだサポートされていますか?**

はい、管理ディスクと非管理ディスクの両方をサポートしています。 ただし、新しいワークロードでは管理ディスクの使用を開始し、Managed Disks に現在のワークロードを移行することをお勧めします。

**サイズが 128 GB のディスクを作成した後で 130 GB に増やした場合は、1 つ上のディスク サイズ (512 GB) として課金されますか?**

はい。

**LRS、GRS、および ZRS の Managed Disks を作成できますか?**

現在、Azure Managed Disks ではローカル冗長ストレージ (LRS) しかサポートされていません。

## <a name="premium-disks--both-managed-and-unmanaged"></a>Premium ディスク – 管理ディスクと非管理ディスク

**VM が使用するサイズ シリーズが Premium ストレージ (DSv2 など) をサポートしている場合、Premium データ ディスクと Standard データ ディスクの両方を接続できますか?** 

はい。

**Premium と Standard 両方のデータ ディスクを、Premium Storage をサポートしないサイズ シリーズ (D、Dv2、G、F シリーズなど) に接続できますか?**

いいえ。 Premium Storage をサポートするサイズ シリーズを使用しない VM に接続できるのは、Standard データ ディスクのみです。

**Premium データ ディスクをサイズが 80 GB の既存の VHD から作成した場合、どのくらいの料金がかかりますか?**

80 GB の VHD から作成した Premium データ ディスクは、1 つ上の Premium ディスク サイズすなわち P10 ディスクとして扱われます。 従って、P10 ディスクの価格が課金されます。

**Premium Storage の使用ではトランザクションのコストは発生しますか?**

プロビジョニングされたディスク サイズごとに固定料金が設定され、IOPS とスループットが制限されます。 それ以外にかかるコストは、送信帯域幅とスナップショットの容量です (該当する場合)。 詳細については[価格のページ](https://azure.microsoft.com/pricing/details/storage)をご覧ください。

**ディスク キャッシュの IOPS とスループットの制限は何ですか?**

DS シリーズのキャッシュとローカル SSD の制限の合計は、コアあたり 4000 IOPS、またコアあたり 33 MB/秒 です。 GS シリーズでは、コアあたり 5000 IOP、コアあたり 50 MB/秒です。

**ローカル SSD は Managed Disks VM でサポートされていますか?**

ローカル SSD とは、Managed Disks VM に含まれている一時的なストレージです。 この一時ストレージに追加の料金は発生しません。 これは Azure Blob Storage に永続化されないため、アプリケーション データの保存にこのローカル SSD を使用しないことをお勧めします。

## <a name="what-if-my-question-isnt-answered-here"></a>ここに質問の答えがない場合はどうすればいいですか。

質問がここに表示されていない場合はご連絡ください。答えを見つけるお手伝いをします。 この記事に関する質問は、この記事の末尾のコメント欄、または MSDN [Azure Storage フォーラム](https://social.msdn.microsoft.com/forums/azure/home?forum=windowsazuredata)に投稿して、Azure Storage チームや他のコミュニティ メンバーと意見を交わすことができます。

機能についてのご要望がある場合は、ご要望やアイデアを [Azure Storage フィードバック フォーラム](https://feedback.azure.com/forums/217298-storage)までお送りください。


<!--HONumber=Feb17_HO2-->


