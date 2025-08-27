---
title: Migrate ExpressRoute Gateway Basic SKU public IP to Standard SKU public IP
description: Short guidance for Azure Dedicated HSM customers who need to change an ExpressRoute Gateway (ERGW) Basic SKU public IP to a Standard SKU public IP — high-level steps only.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 08/27/2025
services: dedicated-hsm
ms.service: azure-dedicated-hsm
---

# Migrate ExpressRoute Gateway Basic SKU public IP to Standard SKU public IP

There is no supported in-place migration to change an ExpressRoute Gateway (ERGW) Basic SKU public IP to a Standard SKU public IP for Azure Dedicated HSM. The supported approach is simple and intentionally light on operational detail — provision a new VNet and ERGW (Standard SKU), deploy a new Dedicated HSM (or HSMs for HA) into that VNet, migrate keys as appropriate, validate, then decommission the old HSM.

> [!WARNING]
> Changing Thales Luna partition policies (for example to enable key export) is destructive and will zeroize HSM contents. For details on cloning/export restrictions and partition policy behavior, see [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md).

High-level migration steps

- Provision a new VNet and ExpressRoute Gateway with a Standard SKU public IP.
- Deploy new Dedicated HSM device(s) into the new VNet and configure them in your existing HA topology.
- Migrate keys: clone keys only if partition policies and HA configuration permit; otherwise create new keys in the new HSM(s).
- Validate applications against the new HSM(s), then decommission the old HSM following standard deprovisioning guidance.

If you are not comfortable with any step above, consult the detailed guidance in [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md) or open a support ticket.