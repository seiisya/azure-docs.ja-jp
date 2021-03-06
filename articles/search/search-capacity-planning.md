---
title: クエリとインデックスのワークロードの容量を調整する
titleSuffix: Azure Cognitive Search
description: Azure Cognitive Search のパーティションとレプリカのコンピューティング リソースを調整します。各リソースは課金対象の検索単位で価格設定されます。
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 02/14/2020
ms.openlocfilehash: e2ba5301b81b1a6f5de696ab4587cd8ff43e3c68
ms.sourcegitcommit: 6ee876c800da7a14464d276cd726a49b504c45c5
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 02/19/2020
ms.locfileid: "77462566"
---
# <a name="adjust-capacity-in-azure-cognitive-search"></a>Azure Cognitive Search の容量を調整する

[検索サービスをプロビジョニング](search-create-service-portal.md)し、特定の価格レベルをロックする前に、サービス内のレプリカとパーティションの役割、およびリソース需要の急増と急減に対応するためのサービスの調整方法について理解しておいてください。

容量は、[選択するレベル](search-sku-tier.md)(レベルによってハードウェア特性を決定)、および予測されるワークロードに必要なレプリカとパーティションの組み合わせによって決まります。 調整のレベルとサイズによっては、容量の追加または削減に15分から数時間かかることがあります。 

レプリカやパーティションの割り当てを変更する場合は、Azure Portal の使用をお勧めします。 ポータルでは、階層の上限に達しないようにするための許容される組み合わせに強制的に制限されます。 ただし、スクリプト ベースまたはコード ベースのプロビジョニング方法が必要な場合は、代わりに [Azure PowerShell](search-manage-powershell.md) または[管理 REST API](https://docs.microsoft.com/rest/api/searchmanagement/services) を使用します。

## <a name="terminology-replicas-and-partitions"></a>用語: レプリカとパーティション

|||
|-|-|
|*パーティション* | 読み取り/書き込み操作 (たとえば、インデックスを再構築または更新する場合など) のためにインデックスのストレージと I/O を提供します。 各パーティションには、インデックス全体の共有があります。 3 つのパーティションを割り当てると、インデックスは 3 つに分割されます。 |
|*レプリカ* | 主にクエリ操作の負荷分散に使用される Search サービスのインスタンスです。 各レプリカは、インデックスの 1 つのコピーです。 3 つのレプリカを割り当てると、クエリ要求のサービスに使用できるインデックスのコピーが3つ作成されます。|

## <a name="when-to-add-nodes"></a>ノードを追加するタイミング

最初に、1 つのパーティションと 1 つのレプリカで構成される最小限のリソースがサービスに割り当てられます。 

1 つのサービスに、すべてのワークロード (インデックスおよびクエリ) を処理するための十分なリソースが必要です。 どちらのワークロードもバックグラウンドで実行されません。 クエリ要求の頻度が低い時間帯にインデックスを作成するようにスケジュールできますが、それ以外の時間帯では、サービスはあるタスクを別のタスクより優先させません。 さらに、ある程度の冗長性により、サービスまたはノードが内部的に更新されるときのクエリのパフォーマンスの問題を解決します。

一般的な規則として、検索アプリケーションでは、パーティションよりもレプリカの方が多く必要となる傾向があります。特に、サービス操作でクエリ ワークロードの比重が高い場合は、その傾向が強まります。 その理由については、[高可用性](#HA)に関するセクションで説明します。

レプリカまたはパーティションを追加すると、サービスの実行コストが増加します。 ノードを追加した場合の課金の影響を理解するには、[料金計算ツール](https://azure.microsoft.com/pricing/calculator/) を必ず確認してください。 [下のグラフ](#chart)は、特定の構成に必要な検索単位の数を相互参照するのに役立ちます。

## <a name="how-to-allocate-replicas-and-partitions"></a>レプリカとパーティションを割り当てる方法

1. [Azure Portal](https://portal.azure.com/) にサインインし、Search サービスを選択します。

1. **[設定]** で、 **[スケール]** ページを開いてレプリカとパーティションの数を変更します。 

   次のスクリーンショットは、1 つのレプリカとパーティションでプロビジョニングされる標準のサービスを示しています。 下部の式は、使用される検索ユニットの数 (1) を示しています。 ユニットの価格が 100 ドル (実際の価格ではありません) の場合、このサービスを実行するための毎月のコストは平均 100 ドルになります。

   ![現在の値が表示されている [スケール] ページ](media/search-capacity-planning/1-initial-values.png "現在の値が表示されている [スケール] ページ")

1. スライダーを使ってパーティションの数を増減します。 下部の式は、使用される検索ユニットの数を示しています。

   この例では、レプリカとパーティションがそれぞれ 2 つに設定されており、容量が 2 倍になります。 請求の計算式では、レプリカ数とパーティション数が乗算されるため (2 x 2)、検索ユニット数が 4 になっていることに注意してください。 容量を 2 倍にすると、サービスを実行するためのコストが 2 倍以上になります。 検索ユニットのコストが 100 ドルの場合、新しい毎月の請求額は 400 ドルになります。

   各レベルの現在のユニットあたりのコストについては、[価格のページ](https://azure.microsoft.com/pricing/details/search/)をご覧ください。

   ![レプリカとパーティションの追加](media/search-capacity-planning/2-add-2-each.png "レプリカとパーティションの追加")

1. **[保存]** をクリックして変更を確認します。

   ![スケールと請求の変更の確認](media/search-capacity-planning/3-save-confirm.png "スケールと請求の変更の確認")

   容量の変更が完了するまでに数時間かかります。 プロセスが開始されたらキャンセルすることはできません。また、レプリカとパーティションの調整のリアルタイムの監視はありません。 ただし、変更が行われている間、次のメッセージが常に表示されています。

   ![ポータルのステータス メッセージ](media/search-capacity-planning/4-updating.png "ポータルのステータス メッセージ")

> [!NOTE]
> サービスをプロビジョニングした後は、上位のレベルにアップグレードすることはできません。 新しいレベルで Search Service を作成し、インデックスを再度読み込む必要があります。 サービス プロビジョニングについては、[ポータルで Azure Cognitive Search サービスの作成](search-create-service-portal.md)に関する記事を参照してください。
>
> さらに、パーティションとレプリカは、サービスによって排他的に管理されます。 プロセッサの関係や、ワークロードを特定のノードに割り当てる概念はありません。
>

<a id="chart"></a>

## <a name="partition-and-replica-combinations"></a>パーティションとレプリカの組み合わせ

Basic サービスは、3 SU の上限に対して厳密に 1 個のパーティションと最大 3 個のレプリカを持つことができます。 調整可能なリソースはレプリカだけです。 クエリの高可用性のためには、少なくとも 2 個のレプリカが必要です。

すべての Standard および Storage Optimized 検索サービスでは、36 SU の制限のもとで、レプリカとパーティションの次の組み合わせを想定できます。 

|   | **1 個のパーティション** | **2 個のパーティション** | **3 個のパーティション** | **4 個のパーティション** | **6 個のパーティション** | **12 個のパーティション** |
| --- | --- | --- | --- | --- | --- | --- |
| **1 つのレプリカ** |1 SU |2 SU |3 SU |4 SU |6 SU |12 SU |
| **2 つのレプリカ** |2 SU |4 SU |6 SU |8 SU |12 SU |24 SU |
| **3 つのレプリカ** |3 SU |6 SU |9 SU |12 SU |18 SU |36 SU |
| **4 つのレプリカ** |4 SU |8 SU |12 SU |16 SU |24 SU |該当なし |
| **5 つのレプリカ** |5 SU |10 SU |15 SU |20 SU |30 SU |該当なし |
| **6 つのレプリカ** |6 SU |12 SU |18 SU |24 SU |36 SU |該当なし |
| **12 レプリカ** |12 SU |24 SU |36 SU |該当なし |該当なし |該当なし |

SU、価格、および容量の詳細については、Azure Web サイトをご覧ください。 詳細については、「[料金の詳細](https://azure.microsoft.com/pricing/details/search/)」をご覧ください。

> [!NOTE]
> レプリカとパーティションの数は、均等に 12 分割されます (具体的には、1、2、3、4、6、12)。 これは、Azure Cognitive Search では各インデックスがすべてのパーティションに均等に分散されるように 12 のシャードに事前に分割されるためです。 たとえば、サービスに 3 つのパーティションがあり、インデックスを作成する場合、各パーティションにはインデックスの 4 つのシャードを含めます。 Azure Cognitive Search でインデックスがどのようにシャードされるかは実装の問題であり、今後のリリースで変更される場合があります。 今日 12 個であっても、今後も必ず 12 個になるとは限りません。
>

<a id="HA"></a>

## <a name="high-availability"></a>高可用性

通常、簡単かつ比較的速くスケールアップできるため、1 つのパーティションと 1 ～ 2 つのレプリカで開始し、クエリ数の増加に合わせてスケールアップすることをお勧めします。 クエリのワークロードは主にレプリカ上で実行されます。 より高いスループットまたは高可用性のためには、レプリカの追加が必要となります。

高可用性のための一般的な推奨事項は次のとおりです。

* 読み取り専用ワークロード (クエリ) の高可用性を実現するには 2 つのレプリカ

* 読み取り/書き込みワークロード (クエリに加え、個々のドキュメントを追加、更新、または削除するときのインデックス作成) の高可用性を実現するには 3 つ以上のレプリカ

Azure Cognitive Search のサービス レベル アグリーメント (SLA) は、クエリ操作と、ドキュメントの追加、更新、削除から成るインデックスの更新とを対象としています。

Basic レベルでは、1 つのパーティションと 3 つのレプリカが上限です。 インデックス作成とクエリのスループットに対する需要の変動にすばやく反応する柔軟性が必要な場合は、Standard レベルのいずれかを検討してください。  ストレージ要件がクエリ スループットよりもはるかに速く増大している場合は、Storage Optimized レベルの 1 つを検討します。

## <a name="disaster-recovery"></a>障害復旧

現時点では、障害復旧のための組み込みのメカニズムはありません。 追加のパーティションまたはレプリカは、ディザスター リカバリーの目標を達成する手段として適切ではありません。 最も一般的なアプローチでは、別のリージョンにもう 1 つの検索サービスを設定することで、そのサービス レベルに冗長性を追加します。 インデックスの再構築中の可用性の場合と同様に、リダイレクトまたはフェールオーバーのロジックをコードに用意する必要があります。

## <a name="estimate-replicas"></a>レプリカの推定

生産サービスでは、SLA のために 3 つのレプリカを割り当てる必要があります。 クエリ パフォーマンスを遅くすると、インデックスの追加のコピーがオンラインになって、より多くのクエリ ワークロードに対応し、複数のレプリカ全体の要求を負荷分散できるようになります。

クエリの負荷に対応するために必要なレプリカの数に関するガイドラインは提供されていません。 クエリのパフォーマンスは、クエリおよび競合するワークロードの複雑さによって異なります。 レプリカを追加するとパフォーマンスが向上しますが、厳密に線形に向上するわけではありません。3 つのレプリカを追加しても、スループットが 3 倍になるとは限りません。

ソリューションの QPS を推定する方法については、「[パフォーマンスのスケール](search-performance-optimization.md)および[クエリの監視 ](search-monitor-queries.md)」を参照してください。

## <a name="estimate-partitions"></a>パーティションの推定

[選択するレベル](search-sku-tier.md)によってパーティションのサイズと速度が決まり、各レベルはさまざまなシナリオに適した一連の特性に対して最適化されます。 ハイエンド レベルを選択した場合は、S1 を使用する場合よりもパーティション数を減らす必要がある場合があります。 自己指示型のテストを通じて回答する必要がある質問の 1 つは、より大きなパーティションの方が、低いレベルでプロビジョニングされたサービス上の 2 つの安価なパーティションよりもパフォーマンスが優れているかどうかということです。

ほぼリアルタイムのデータ更新を要求する検索アプリケーションでは、レプリカよりも多くのパーティションが比例して必要になります。 パーティションを追加すると、読み取り/書き込み操作がより多くのコンピューティング リソースに分散されます。 また、追加のインデックスとドキュメントを格納するためのディスク領域も増加します。

インデックスが大きくなると、クエリの実行に時間がかかります。 そのため、パーティションで段階的な増加が発生するたびに、レプリカでは小規模であるが比例的な増加が必要となる場合があります。 クエリの複雑さとボリュームは、クエリの実行が完了するまでの速度の要因となります。

## <a name="next-steps"></a>次のステップ

> [!div class="nextstepaction"]
> [Azure Cognitive Search の価格レベルの選択](search-sku-tier.md)