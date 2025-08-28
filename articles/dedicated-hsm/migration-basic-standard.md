---
title: Migrate ExpressRoute Gateway Basic SKU public IP to Standard SKU public IP
description: Short guidance for Azure Dedicated HSM customers who need to change an ExpressRoute Gateway (ERGW) Basic SKU public IP to a Standard SKU public IP.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 08/27/2025
services: dedicated-hsm
ms.service: azure-dedicated-hsm
---

# Migrate ExpressRoute Gateway Basic SKU public IP to Standard SKU public IP

There is no smooth migration for Dedicated HSM using ExpressRoute Gateway (ERGW) Basic SKU public IP to Standard SKU public IP. Customer will need to create a new Dedicated HSM with a new VNET/ a new ERGW with Standard SKU public IP, clone keys to new HSM with HA setup, delete the old Dedicated HSM.

If you have any concerns with this, please contact your customer success account manager, or cloud solution architect.

For more information, see:

- [Azure Dedicated HSM overview](overview.md)
- [Azure ExpressRoute documentation](/azure/expressroute/expressroute-about-virtual-network-gateways)
- [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md).
