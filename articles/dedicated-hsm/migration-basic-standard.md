---
title: Migrate Dedicated HSM from ExpressRoute Basic SKU
description: Learn how to migrate Azure Dedicated HSM connectivity from the ExpressRoute gateway Basic SKU to the Standard SKU before retirement.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 07/31/2026
services: dedicated-hsm
ms.service: azure-dedicated-hsm
ms.custom: references_regions
ai-usage: ai-assisted
---

# Migrate Dedicated HSM from ExpressRoute Basic SKU

Microsoft extended support for the Basic SKU ExpressRoute gateway used by Azure Dedicated HSM until March 31, 2027. If your Dedicated HSM deployment uses the Basic SKU, complete the migration before this date to maintain connectivity.

Azure Dedicated HSM uses an ExpressRoute gateway and a service-managed circuit to connect the Dedicated HSM virtual network interface to your virtual network. You can't migrate this gateway by using the standard [ExpressRoute gateway migration process](/azure/expressroute/gateway-migration). The standard process doesn't support a virtual network that contains a Dedicated HSM.

> [!CAUTION]
> Don't change, delete, or re-create the ExpressRoute gateway in an existing Dedicated HSM environment. These actions disrupt the service-managed circuit and make the Dedicated HSM deployment inaccessible.
>
> Don't power down or deallocate a Dedicated HSM to make it eligible for the standard ExpressRoute gateway migration process. You can't power on the device independently. Recovery requires an [Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request) and can significantly extend the outage.

The supported migration path is to deploy a new Dedicated HSM environment that uses a Standard SKU ExpressRoute gateway, establish high availability between the existing and new environments, and then move your workloads.

Microsoft doesn't have access to your allocated Dedicated HSM devices, key material, or workload automation. You're responsible for planning and completing the migration.

> [!IMPORTANT]
> Azure Dedicated HSM retires on July 31, 2028. Consider migrating directly to [Azure Cloud HSM](../cloud-hsm/overview.md) or [Azure Key Vault Managed HSM](../key-vault/managed-hsm/overview.md).
>
> If you can't complete that service migration before March 31, 2027, first migrate your Dedicated HSM environment to a Standard SKU ExpressRoute gateway. Then complete your service migration before the Dedicated HSM retirement date.

## Prepare for the migration

Before you begin:

- Inventory applications and workloads that connect to the existing Dedicated HSM environment.
- Confirm that you can deploy the required Dedicated HSM capacity in the target region. You might need to [request a quota increase](troubleshoot.md#deployment-failure-based-on-quota) for the target subscription before you deploy the new HSMs.
- Plan a maintenance window for application connectivity changes.
- Review your high availability configuration and verify that the existing HSMs are healthy.
- Prepare application and operational validation tests for the new environment.

West Europe, North Europe, East US, East US 2, and West US can have limited Dedicated HSM capacity. In these regions, you might need to use a rolling migration:

1. Deploy one new Dedicated HSM.
1. Add it to the existing high availability group.
1. Validate replication and application connectivity.
1. Zeroize and delete one old HSM to release capacity.
1. Repeat the process until you migrate all HSMs.

Confirm regional capacity and your migration sequence with Azure support before you begin.

## Deploy a new Dedicated HSM environment

Create an environment that uses a Standard SKU ExpressRoute gateway:

1. Create a new virtual network.
1. Create a Standard SKU ExpressRoute gateway.
1. Deploy new Dedicated HSM instances.
1. Configure networking, routing, and application connectivity.
1. Validate connectivity and operational readiness.

Don't modify the ExpressRoute gateway in the existing Dedicated HSM environment.

## Configure high availability

Establish high availability between the existing and new Dedicated HSM environments:

1. Create the required HSM partitions and configurations.
1. Configure the same cloning domain on the existing and new partitions.
1. Add the new partitions to the existing high availability group.
1. Verify synchronization, replication health, and key availability from dependent applications.

For more information, see [Azure Dedicated HSM high availability](high-availability.md).

> [!NOTE]
> Don't continue until both environments can serve production workloads and replication is healthy.

## Migrate workloads

Move one application or workload at a time:

1. Update the application to connect to the new Dedicated HSM environment.
1. Validate cryptographic operations, monitoring, and operational procedures.
1. Monitor the workload for errors or unexpected behavior.
1. Repeat these steps until all production workloads use the new environment.

Keep the existing environment available until you complete application and operational validation.

## Decommission the old environment

After all workloads use the new environment:

1. Confirm that no applications connect to the old HSMs.
1. Verify that the new environment has the required key material and capacity.
1. Follow [How to delete an HSM resource](troubleshoot.md#how-to-delete-an-hsm-resource) to zeroize and delete each old Dedicated HSM.
1. Delete the old Dedicated HSM networking resources only after Azure removes all old HSM resources.

> [!WARNING]
> Don't delete the resource group before you zeroize and delete each Dedicated HSM resource. Deleting the resource group can leave the HSM in an orphaned state while billing continues.

## Why can't I use the standard ExpressRoute migration process?

Dedicated HSM connectivity uses a service-managed ExpressRoute circuit that binds the Dedicated HSM virtual network interface to the ExpressRoute gateway. Standard networking operations can't re-create or migrate this circuit.

Don't take any of the following actions in the existing environment:

- Don't migrate the existing gateway from Basic SKU to Standard SKU.
- Don't modify the ExpressRoute gateway configuration.
- Don't delete or re-create the ExpressRoute gateway.

Deploy a new environment and move your workloads instead of changing the gateway in the existing environment.

## Related content

- [Azure Dedicated HSM overview](overview.md)
- [Azure Dedicated HSM networking](networking.md)
- [Migrate from Azure Dedicated HSM to Azure Key Vault Managed HSM or Azure Cloud HSM](migration-guide.md)
- [About ExpressRoute gateway migration](/azure/expressroute/gateway-migration)
