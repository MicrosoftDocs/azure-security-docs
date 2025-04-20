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
| :------------------- |:---------------:|
|Connectivity to a payment HSM in a local virtual network | Yes |
|Connectivity to a payment HSM in a peered virtual network (Same region) | Yes |
|Connectivity to a payment HSM in a peered virtual network (Cross region or global peering) | No |
|Connectivity to a payment HSM over ExpressRoute gateway | Yes|
|ExpressRoute (ER) FastPath | No |
|Connectivity from on-premises to a payment HSM in a spoke virtual network over ExpressRoute gateway and virtual network peering with gateway transit | Yes |
|Connectivity from on-premises to a payment HSM in a spoke virtual network over VPN gateway | Yes |
|Connectivity from on-premises to a payment HSM in a spoke virtual network over VPN gateway and virtual network peering with gateway transit | Yes |
|Connectivity over Active/Passive VPN gateways | Yes |
|Connectivity over Active/Active VPN gateways | No |
|Connectivity over Active/Active Zone Redundant gateways | No |
|Connectivity over Virtual WAN (VWAN) | No |

## Constraints

The following table describes what is supported for each network features configuration:

|Features |Basic network features |
| :------------------- | -------------------: |
|Delegated subnet per virtual network | 1 |
|[Network Security Groups](/azure/virtual-network/network-security-groups-overview) on payment HSMs on Azure-delegated subnets | No |
|[User-defined routes (UDRs)](/azure/virtual-network/virtual-networks-udr-overview#user-defined) on payment HSMs on Azure-delegated subnets | No |
|Connectivity to [private endpoints](/azure/private-link/private-endpoint-overview) | No |
|Load balancers for payment HSMs on Azure traffic | No |
|Dual stack (IPv4 and IPv6) virtual network | IPv4 only supported |

## Next steps

- [What is Azure Payment HSM?](overview.md)
- [Azure Payment HSM deployment scenarios](deployment-scenarios.md)
- [Azure Payment HSM traffic inspection](inspect-traffic.md)
- [Get started with Azure Payment HSM](getting-started.md)
- [Create a payment HSM](create-payment-hsm.md)
- [Frequently asked questions](faq.yml)
