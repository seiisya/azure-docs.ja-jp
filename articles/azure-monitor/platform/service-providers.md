---
title: サービス プロバイダー向けの Azure Monitor ログ | Microsoft Doc
description: Azure Monitor ログは、マネージド サービス プロバイダー (MSP)、大企業、独立系ソフトウェア ベンダー (ISV)、およびホスティング サービス プロバイダーが、顧客のオンプレミス型またはクラウド型インフラストラクチャのサーバーを管理および監視するのに役立ちます。
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: MeirMen
ms.author: meirm
ms.date: 02/03/2020
ms.openlocfilehash: 1b5c181569ea569ecc4808284683501cb20f7bf5
ms.sourcegitcommit: 7c18afdaf67442eeb537ae3574670541e471463d
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/11/2020
ms.locfileid: "77117067"
---
# <a name="azure-monitor-logs-for-service-providers"></a>サービス プロバイダー向けの Azure Monitor ログ

Azure Monitor の Log Analytics ワークスペースは、マネージド サービス プロバイダー (MSP)、大企業、独立系ソフトウェア ベンダー (ISV)、およびホスティング サービス プロバイダーが、顧客のオンプレミス型またはクラウド型インフラストラクチャのサーバーを管理および監視するのに役立ちます。

特に多くの異なる部署の IT 管理を担当する一元的な IT チームがある大企業では、サービス プロバイダーと多くの類似点を共有します。 このドキュメントでは簡単にするために*サービス プロバイダー* という言葉を使用していますが、企業およびその他のお客様にも同じ機能をご利用いただけます。

[クラウド ソリューション プロバイダー (CSP)](https://partner.microsoft.com/Solutions/cloud-reseller-overview) プログラムに参加しているパートナーやサービス プロバイダーにとって、Azure Monitor の Log Analytics は Azure CSP サブスクリプションで使用できる Azure サービスの 1 つです。

Azure Monitor の Log Analytics は、[Azure Lighthouse](https://docs.microsoft.com/azure/lighthouse/overview) の Azure の委任されたリソース管理機能を使用して顧客のリソースを管理しているサービス プロバイダーが使用することもできます。

## <a name="architectures-for-service-providers"></a>サービス プロバイダー向けアーキテクチャ

Log Analytics ワークスペースは、[ログ](data-platform-logs.md) データのフローと分離を制御し、特定のビジネス ニーズに対処するアーキテクチャを作成するための方法を管理者にもたらします。 [この記事](design-logs-deployment.md)では、ワークスペースの設計、デプロイ、移行に関する考慮事項について説明します。また、[アクセスの管理](manage-access.md)に関する記事では、ログ データへのアクセス許可を適用および管理する方法についても説明します。 サービス プロバイダーには追加の考慮事項があります。

Log Analytics ワークスペースに関するサービス プロバイダー向けのアーキテクチャは次の 3 つ考えられます。

### <a name="1-distributed---logs-are-stored-in-workspaces-located-in-the-customers-tenant"></a>1.分散 - ログは、顧客のテナントに置かれたワークスペースに格納されます

このアーキテクチャでは、ワークスペースは、顧客のテナント (その顧客のすべてのログに使用) にデプロイされます。

サービス プロバイダーの管理者は、次の 2 つの方法で、顧客のテナントにある Log Analytics ワークスペースにアクセスできます。

- 顧客は、サービス プロバイダーから個々のユーザーを [Azure Active Directory のゲスト ユーザー (B2B)](https://docs.microsoft.com/azure/active-directory/b2b/what-is-b2b) として追加できます。 サービス プロバイダーの管理者は、これらのワークスペースにアクセスするために、Azure portal 内の各顧客のディレクトリにサインインする必要があります。 そのためには、顧客がサービス プロバイダーの管理者ごとに個別のアクセスを管理する必要もあります。
- スケーラビリティと柔軟性を高めるために、サービス プロバイダーは [Azure Lighthouse](https://docs.microsoft.com/azure/lighthouse/overview) の [Azure の委任されたリソース管理](https://docs.microsoft.com/azure/lighthouse/concepts/azure-delegated-resource-management)機能を使用して、顧客のテナントにアクセスできます。 この方法では、サービス プロバイダーの管理者はサービス プロバイダーのテナント内の Azure AD ユーザー グループに含まれます。このグループには、各顧客のオンボード プロセス中にアクセス権が付与されます。 これらの管理者はその後、各顧客のテナントに個別にログインするのではなく、独自のサービス プロバイダーのテナント内から各顧客のワークスペースにアクセスできます。 この方法で顧客の Log Analytics ワークスペースのリソースにアクセスすると、顧客側で必要な作業が減り、同じサービス プロバイダーで管理されている複数の顧客を対象にしたデータの収集および分析が [Azure Monitor ブック](https://docs.microsoft.com/azure//azure-monitor/platform/workbooks-overview)などのツールで簡単に行えるようになります。 詳細については、[顧客のリソースの大規模な監視](https://docs.microsoft.com/azure/lighthouse/how-to/monitor-at-scale)に関する記事をご覧ください。

分散アーキテクチャの利点は次のとおりです。

* 顧客は [Azure の委任されたリソース管理](https://docs.microsoft.com/azure/lighthouse/concepts/azure-delegated-resource-management)を通じて特定のレベルのアクセス許可を確認することも、独自の[ロールベースのアクセス](https://docs.microsoft.com/azure/role-based-access-control/overview)を使用してログへのアクセスを管理することもできます。
* エージェントベースの VM データだけでなく、あらゆる種類のリソースからログを収集できます。 たとえば、Azure 監査ログ です。
* 保有期間やデータ キャッピングなど、顧客ごとに異なるワークスペース設定が可能です。
* 規制やコンプライアンスについて顧客間で分離します。
* 各ワークスペースの料金は、顧客のサブスクリプションにまとめられます。

分散アーキテクチャの短所は次のとおりです。

* Azure Monitor ブックなどのツールを使用して顧客の全テナントを対象にデータを一元的に視覚化して分析すると、特に 50 個以上のワークスペースでデータを分析しているときに、処理速度が低下する可能性があります。
* 顧客が Azure の委任されたリソース管理をオンボードしていない場合、サービス プロバイダーの管理者は顧客のディレクトリにプロビジョニングされている必要があり、一度に多数の顧客のテナントをサービス プロバイダーが管理することが困難になります。

### <a name="2-central---logs-are-stored-in-a-workspace-located-in-the-service-provider-tenant"></a>2.中央 - ログは、サービス プロバイダーのテナントに置かれているワークスペースに格納されます

このアーキテクチャでは、ログは、顧客のテナントに格納されませんが、サービス プロバイダーのいずれかのサブスクリプション内にある中央の場所にのみ格納されます。 顧客の VM 上にインストールされたエージェントは、ワークスペース ID と秘密鍵を使用してこのワークスペースにログを送信するように構成されています。

中央アーキテクチャの利点は次のとおりです。

* 多数の顧客を管理して、さまざまなバックエンド システムに統合するときに簡単に行えます。
* サービス プロバイダーは、ログと、関数や保存クエリなどの成果物に対する完全な所有権を持ちます。
* サービス プロバイダーは、すべての顧客の分析を実行できます。

中央アーキテクチャの短所は次のとおりです。

* このアーキテクチャは、エージェント ベースの VM データにのみ適用可能で、PaaS、SaaS、Azure ファブリック データ ソースには対応しません。
* 顧客間のデータは、単一のワークスペースにマージされると、分離することが困難になる可能性があります。 これを行う唯一の適切な方法は、コンピューターの完全修飾ドメイン名 (FQDN) を使用するか、または Azure サブスクリプション ID を介するというものです。 
* すべてのお客様からのすべてのデータは、単一の請求書と、同じ保有期間および構成設定を伴う同じリージョンに格納されます。
* Microsoft Azure Diagnostics や Azure 監査ログなどの Azure のファブリックおよび PaaS サービスは、ワークスペースがリソースと同じテナントにあることを必要とするため、中央のワークスペースにログを送信することができません。
* すべての顧客からのすべての VM エージェントは、同じワークスペース ID とキーを使用して中央のワークスペースで認証されます。 他の顧客を中断せずに特定の顧客からのログをブロックする方法はありません。

### <a name="3-hybrid---logs-are-stored-in-workspace-located-in-the-customers-tenant-and-some-of-them-are-pulled-to-a-central-location"></a>3.ハイブリッド - ログは、顧客のテナントに置かれているワークスペースに格納され、その一部は中央の場所にプルされます。

3 番目のアーキテクチャは、2 つのオプションを組み合わせたものです。 これは、ログが各顧客の元でローカルに格納されるという最初の分散アーキテクチャに基づきながら、何らかのメカニズムを使用して、ログの中央のリポジトリを作成します。 ログの一部はレポートと分析のために中央の場所にプルされます。 この一部は、少数のデータ型であることも、日次統計などのアクティビティの概要であることもあります。

ログを中央の場所に実装するには 2 つの方法があります。

1. 中央ワークスペース:サービス プロバイダーでは、そのテナント内にワークスペースを作成し、[クエリ API](https://dev.loganalytics.io/) を[データ収集 API](../../azure-monitor/platform/data-collector-api.md) と共に利用して、さまざまなワークスペースからこの中央の場所にデータを移動させるスクリプトを使用できます。 スクリプト以外のもう 1 つのオプションは [Azure Logic Apps](https://docs.microsoft.com/azure/logic-apps/logic-apps-overview) を使用するというものです。

2. 中央の場所としての Power BI:Power BI は、さまざまなワークスペースで Log Analytics ワークスペースと [Power BI](../../azure-monitor/platform/powerbi.md) との統合を利用してデータをエクスポートするときに、中央の場所として機能します。 

## <a name="next-steps"></a>次のステップ

* [Resource Manager テンプレート](template-workspace-configuration.md)を使用してワークスペースの作成および構成を自動化する

* [PowerShell](../../azure-monitor/platform/powershell-workspace-configuration.md) を使用してワークスペースの作成を自動化する 

* [アラート](../../azure-monitor/platform/alerts-overview.md) を使用して既存のシステムと統合する

* [Power BI](../../azure-monitor/platform/powerbi.md) を使用して概要レポートを作成する

* [Azure の委任されたリソース管理](https://docs.microsoft.com/azure/lighthouse/concepts/azure-delegated-resource-management)に顧客をオンボードする
