---
title: インクルード ファイル
titleSuffix: Azure
description: インクルード ファイル
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: badba278e88e5065d8a4ff57ca9506fe1f9ad203
ms.sourcegitcommit: aee08b05a4e72b192a6e62a8fb581a7b08b9c02a
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 01/09/2020
ms.locfileid: "75773634"
---
1. 選択した Direct ピアリングで接続を表示します。
    ```powershell
    $directPeering.Connections

    PeeringDBFacilityId         : 6
    UseForPeeringService        : False
    SessionAddressProvider      : Peer
    SessionPrefixV4             : 12.245.179.20/30
    ConnectionIdentifier        : 3f196dfe-c174-4900-ab1c-fb7b50ad1535
    SessionPrefixV6             :
    MicrosoftSessionIPv4Address : 12.245.179.22
    BandwidthInMbps             : 100000
    SessionStateV4              : Established
    SessionStateV6              : None
    ConnectionState             : Active
    ```
1. Peering Service に対して有効にする接続を選択します。 この例では、使用できる唯一の接続を使用します。
    ```powershell
    $directPeering.Connections[1] = $directPeering.Connections[1] | Set-AzPeeringDirectConnectionObject -UseForPeeringService $true

    PeeringDBFacilityId         : 6
    UseForPeeringService        : True
    SessionAddressProvider      : Peer
    SessionPrefixV4             : 12.245.179.20/30
    ConnectionIdentifier        : 3f196dfe-c174-4900-ab1c-fb7b50ad1535
    SessionPrefixV6             :
    MicrosoftSessionIPv4Address : 12.245.179.22
    BandwidthInMbps             : 100000
    SessionStateV4              : Established
    SessionStateV6              : None
    ConnectionState             : Active
    ```
1. ここで、以下のコマンドを使用して Direct ピアリングに対する変更を保存します。
    ```powershell
    $directPeering | Update-AzPeering
    ```
    
    出力例を次に示します。
    
    ```powershell
        Name                 : SeattleDirectPeering
        Sku.Name             : Basic_Premium_Free
        Kind                 : Direct
        Connections          : {71,71}
        PeerAsn.Id           : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/SeattleDirectPeering
        UseForPeeringService : True
        PeeringLocation      : Seattle
        ProvisioningState    : Succeeded
        Location             : centralus
        Id                   : /subscriptions/{subscriptionId}/resourceGroups/PeeringResourceGroup/providers/Microsoft.Peering/peerings/SeattleDirectPeering
        Type                 : Microsoft.Peering/peerings
        Tags                 : {}
    ```
    