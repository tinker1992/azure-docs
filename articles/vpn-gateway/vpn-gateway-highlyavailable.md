---
title: 'About Highly Available gateway configurations'
titleSuffix: Azure VPN Gateway
description: Learn about highly available configuration options using Azure VPN Gateways.
author: cherylmc
ms.service: vpn-gateway
ms.topic: article
ms.date: 07/11/2024
ms.author: cherylmc

---
# Highly Available cross-premises and VNet-to-VNet connectivity

This article provides an overview of Highly Available configuration options for your cross-premises and VNet-to-VNet connectivity using Azure VPN gateways.

## <a name = "activestandby"></a>About VPN gateway redundancy

Every Azure VPN gateway consists of two instances in an active-standby configuration. For any planned maintenance or unplanned disruption that happens to the active instance, the standby instance would take over (failover) automatically, and resume the S2S VPN or VNet-to-VNet connections. The switch over will cause a brief interruption. For planned maintenance, the connectivity should be restored within 10 to 15 seconds. For unplanned issues, the connection recovery is longer, about 1 to 3 minutes in the worst case. For P2S VPN client connections to the gateway, the P2S connections are disconnected and the users need to reconnect from the client machines.

:::image type="content" source="./media/vpn-gateway-highlyavailable/active-standby.png" alt-text="Diagram shows an on-premises site with private I P subnets and on-premises V P N connected to an active Azure V P N gateway to connect to subnets hosted in Azure, with a standby gateway available.":::

## Highly Available cross-premises

To provide better availability for your cross premises connections, there are a few options available:

* Multiple on-premises VPN devices
* Active-active Azure VPN gateway
* Combination of both

### <a name = "activeactiveonprem"></a>Multiple on-premises VPN devices

You can use multiple VPN devices from your on-premises network to connect to your Azure VPN gateway, as shown in the following diagram:

:::image type="content" source="./media/vpn-gateway-highlyavailable/multiple-onprem-vpns.png" alt-text="Diagram shows multiple on-premises sites with private I P subnets and on-premises V P N connected to an active Azure V P N gateway to connect to subnets hosted in Azure, with a standby gateway available.":::

This configuration provides multiple active tunnels from the same Azure VPN gateway to your on-premises devices in the same location. There are some requirements and constraints:

1. You need to create multiple S2S VPN connections from your VPN devices to Azure. When you connect multiple VPN devices from the same on-premises network to Azure, you need to create one local network gateway for each VPN device, and one connection from your Azure VPN gateway to each local network gateway.
1. The local network gateways corresponding to your VPN devices must have unique public IP addresses in the "GatewayIpAddress" property.
1. BGP is required for this configuration. Each local network gateway representing a VPN device must have a unique BGP peer IP address specified in the "BgpPeerIpAddress" property.
1. You should use BGP to advertise the same prefixes of the same on-premises network prefixes to your Azure VPN gateway, and the traffic will be forwarded through these tunnels simultaneously.
1. You must use Equal-cost multi-path routing (ECMP).
1. Each connection is counted against the maximum number of tunnels for your Azure VPN gateway. See the [VPN Gateway settings](vpn-gateway-about-vpn-gateway-settings.md#gwsku) page for the latest information about tunnels, connections, and throughput.

In this configuration, the Azure VPN gateway is still in active-standby mode, so the same failover behavior and brief interruption will still happen as described [above](#activestandby). But this setup guards against failures or interruptions on your on-premises network and VPN devices.

### Active-active VPN gateways

You can create an Azure VPN gateway in an active-active configuration, where both instances of the gateway VMs establish S2S VPN tunnels to your on-premises VPN device, as shown the following diagram:

:::image type="content" source="./media/vpn-gateway-highlyavailable/active-active.png" alt-text="Diagram shows an on-premises site with private I P subnets and on-premises V P N connected to two active Azure V P N gateway to connect to subnets hosted in Azure.":::

[!INCLUDE [active-active gateways](../../includes/vpn-gateway-active-active-gateway-include.md)]

### Dual-redundancy: active-active VPN gateways for both Azure and on-premises networks

The most reliable option is to combine the active-active gateways on both your network and Azure, as shown in the following diagram.

:::image type="content" source="./media/vpn-gateway-highlyavailable/dual-redundancy.png" alt-text="Diagram shows a Dual Redundancy scenario.":::

Here you create and set up the Azure VPN gateway in an active-active configuration, and create two local network gateways and two connections for your two on-premises VPN devices as described above. The result is a full mesh connectivity of 4 IPsec tunnels between your Azure virtual network and your on-premises network.

All gateways and tunnels are active from the Azure side, so the traffic is spread among all 4 tunnels simultaneously, although each TCP or UDP flow will again follow the same tunnel or path from the Azure side. Even though by spreading the traffic, you may see slightly better throughput over the IPsec tunnels, the primary goal of this configuration is for high availability. And due to the statistical nature of the spreading, it's difficult to provide the measurement on how different application traffic conditions will affect the aggregate throughput.

This topology requires two local network gateways and two connections to support the pair of on-premises VPN devices, and BGP is required to allow the two connections to the same on-premises network. These requirements are the same as the [above](#activeactiveonprem). 

## Highly Available VNet-to-VNet

The same active-active configuration can also apply to Azure VNet-to-VNet connections. You can create active-active VPN gateways for both virtual networks, and connect them together to form the same full mesh connectivity of 4 tunnels between the two VNets, as shown in the following diagram:

:::image type="content" source="./media/vpn-gateway-highlyavailable/vnet-to-vnet.png" alt-text="Diagram shows two Azure regions hosting private I P subnets and two Azure V P N gateways through which the two virtual sites connect.":::

This ensures there are always a pair of tunnels between the two virtual networks for any planned maintenance events, providing even better availability. Even though the same topology for cross-premises connectivity requires two connections, the VNet-to-VNet topology shown above will need only one connection for each gateway. Additionally, BGP is optional unless transit routing over the VNet-to-VNet connection is required.

## Next steps
See [Configure active-active gateways](active-active-portal.md) using the [Azure portal](active-active-portal.md) or [PowerShell](vpn-gateway-activeactive-rm-powershell.md).

