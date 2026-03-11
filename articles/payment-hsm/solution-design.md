---
title: Solution design for Azure Payment HSM
description: Learn about topologies and constraints for Azure Payment HSM
services: payment-hsm
author: msmbaldwin

ms.service: azure-payment-hsm
ms.topic: article
ms.date: 04/10/2025
ms.author: mbaldwin
---

# Azure Payment HSM solution design

This article identifies topologies and constraints for Azure Payment HSM.

## Supported topologies

The following table describes the network topologies supported by each network features configuration of Azure Payment HSM.

|Topology |Basic network features |
| :------------------------------------------------------------- |:---------------:|
|Connectivity to a payment HSM in a local virtual network | Yes |
|Connectivity to a payment HSM in a peered virtual network (Same region) | Yes |
|Connectivity to a payment HSM in a peered virtual network (Cross region or global peering)<br><br>For connectivity to a payment HSM in a peered virtual network (Cross region or global peering), use firewall/NVA with SNAT or reverse proxy as a workaround. See sample deployment scenario 2 (cross region connection setup with HSM) at [Use your browser to access the payShield manager for your Azure Payment HSM](access-payshield-manager.md). | No |
|Connectivity to a payment HSM over ExpressRoute gateway | Yes |
|ExpressRoute (ER) FastPath | No |
|Connectivity from on-premises to a payment HSM in a spoke virtual network over ExpressRoute gateway and virtual network peering with gateway transit | Yes |
|Connectivity from on-premises to a payment HSM in a spoke virtual network over VPN gateway | Yes |
|Connectivity from on-premises to a payment HSM in a spoke virtual network over VPN gateway and virtual network peering with gateway transit | Yes |
|Connectivity over Active/Passive VPN gateways | Yes |
|Connectivity over Active/Active VPN gateways | No |
|Connectivity over Active/Active Zone Redundant gateways | No |
|Transit connectivity via a virtual WAN for a payment HSM provisioned in a spoke virtual network | Yes |
|Enables traffic inspection via virtual WAN securedHub | Yes |
|On-premises connectivity via virtual WAN attached SD-WAN | No |

> [!NOTE]
> For Virtual WAN (VWAN) securedHub:  
> - With Routing Intent: Add the delegated subnet prefix (x.x.x.x/27) to the Routing Intent's list of prefixes.  
> - Without Routing Intent: Add a route to the VWAN's route table for x.x.x.x/27 and point the next hop to the firewall.

## Constraints

The following table describes what is supported for each network features configuration:

|Features |Basic network features |
| :------------------------------------------------------------- | -------------------: |
|Delegated subnet per virtual network | 1 |
|[Network Security Groups](/azure/virtual-network/network-security-groups-overview) on payment HSMs on Azure-delegated subnets | No |
|[User-defined routes (UDRs)](/azure/virtual-network/virtual-networks-udr-overview#user-defined) on payment HSMs on Azure-delegated subnets | No |
|Connectivity to [private endpoints](/azure/private-link/private-endpoint-overview) in the same virtual network on payment HSM delegated subnets | No |
|Connectivity to [private endpoints](/azure/private-link/private-endpoint-overview) in a different spoke VNet connected to virtual WAN | Yes |
|Load balancers for payment HSMs on Azure traffic | No |
|Dual stack (IPv4 and IPv6) virtual network | IPv4 only supported |

## Next steps

- [What is Azure Payment HSM?](overview.md)
- [Azure Payment HSM deployment scenarios](deployment-scenarios.md)
- [Azure Payment HSM traffic inspection](inspect-traffic.md)
- [Get started with Azure Payment HSM](getting-started.md)
- [Create a payment HSM](create-payment-hsm.md)
- [Frequently asked questions](faq.yml)
