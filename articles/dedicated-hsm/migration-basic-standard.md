---
title: Migrate ExpressRoute Gateway Basic SKU public IP to Standard SKU public IP
description: Guidance for Azure Dedicated HSM customers who need to move from an ExpressRoute Gateway (ERGW) Basic SKU public IP to a Standard SKU public IP. Explains why an in-place change isn't supported and provides a recommended migration approach.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 08/27/2025
services: dedicated-hsm
ms.service: azure-dedicated-hsm
---

# Migrate ExpressRoute Gateway Basic SKU public IP to Standard SKU public IP

There is no supported, smooth in-place migration path for Azure Dedicated HSM deployments that require changing an ExpressRoute Gateway (ERGW) Basic SKU public IP to a Standard SKU public IP. ERGW is bound to the virtual network that hosts your HSMs (a VNet supports one gateway), Dedicated HSMs require a delegated subnet, and Thales Luna partition policies can restrict key export. As a result, the supported approach is to provision a new VNet and ERGW (Standard SKU), deploy new Dedicated HSM(s), migrate keys (clone if partition policies allow, or recreate otherwise), validate, then decommission the old HSM. See [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md) for additional detail on Thales partition policies, cloning, and export restrictions.

> [!WARNING]
> Attempting to change ERGW SKU in-place or changing partition policies on an existing Thales Luna HSM to enable key export can be destructive and may result in lost key material or extended service disruption. Plan carefully, follow the steps below, and coordinate with your internal teams and with Thales/Microsoft support as needed.

## Pre-migration checklist

- Inventory all keys, certificates, and HSM-backed objects to be migrated. Record metadata (key name, purpose, algorithm, policies, expiration, and usage patterns).
- Verify quota and capacity for additional Dedicated HSMs in the chosen region (see quota and capacity guidance in the troubleshooting article).
- Confirm whether your current HSM partitions and HA group permit cloning of key material. If cloning/export is disabled, you will need to recreate keys in the new HSM.
- Identify application owners and plan a maintenance window for the cutover.
- Prepare a rollback plan in case validation fails after cutover.

> [!NOTE]
> To check partition policies on a Thales Luna HSM (for example, whether key export is allowed), run the appropriate Thales Luna administration command such as `partition showPolicies`. See [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md) for details and links to Thales documentation.

## Migration steps (high level)

1. Plan your new network and gateway
   - Decide on the new VNet address space, subnets, and transit hub design. Ensure the new VNet does not conflict with existing address spaces.
   - Provision a new ExpressRoute gateway with the Standard SKU public IP in the new VNet. Refer to the Dedicated HSM networking guidance: [Azure Dedicated HSM networking](networking.md) and the ExpressRoute Gateway documentation.

2. Provision new Dedicated HSM resources
   - Create a delegated subnet in the new VNet for the Dedicated HSMs. Ensure the subnet has the required delegation to `Microsoft.HardwareSecurityModules/dedicatedHSMs`.
   - Deploy one or more Dedicated HSM devices into the new VNet. Use the quickstart guides to provision HSMs by CLI or PowerShell: [Quickstart: Create an Azure Dedicated HSM by using the Azure CLI](quickstart-hsm-azure-cli.md) or [Quickstart: Create an Azure Dedicated HSM with Azure PowerShell](quickstart-create-hsm-powershell.md).

3. Configure High Availability
   - For production scenarios, deploy Dedicated HSMs in an HA configuration per guidance in [Azure Dedicated HSM high availability](high-availability.md) and Thales administration guides. HA ensures resilience during cutover and provides the cloning domain if cloning is supported.

4. Clone or recreate keys
   - If your partitions and HA group are configured to permit cloning, use the Thales-provided cloning workflow to replicate key material from the old HSM to the new HA-enabled HSM cluster. Coordinate cloning operations with Thales support if needed.
   - If cloning or key export is not permitted by the partition policy, you must create new keys in the new HSM and update applications to use them. Key recreation is often the safer option when export is restricted.

5. Validate in the new environment
   - Test application connectivity and cryptographic operations against the new HSM(s). Verify latency, access controls, and availability.
   - Confirm logging, monitoring, and backup processes are configured for the new HSM.

6. Cut over client applications
   - Update client configurations (such as hostname/IP, NTLS/SSL certificates, and client libraries) to point to the new ERGW public IP and HSM private IPs in the new VNet.
   - Perform gradual cutover and validation, or a full maintenance window cutover depending on your risk tolerance.

7. Decommission the old Dedicated HSM
   - Once validation and cutover are complete and you have confirmed the new HSM is handling all production operations, zeroize the old HSM partitions and follow the HSM deprovisioning guidance in the troubleshooting article before deleting the Azure Dedicated HSM resource.

## Thales Luna HSM considerations

Partition policies can prevent export of key material. For complete details about cloning, export restrictions, and the destructive nature of changing partition policies, see [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md).

> [!WARNING]
> Changing partition policies such as the cloning domain or key export after partitions are created is destructive and zeroizes the HSM. This action will permanently remove all key material. If you are unsure of your partition policy state, consult [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md) and Thales documentation, and engage Thales support as needed.

## Troubleshooting and support

- If you encounter deployment failures for new HSMs, verify that the new networking resources are fully provisioned before starting HSM deployment. See troubleshooting guidance for HSM deployment.
- For HA synchronization issues, consult the troubleshooting and high-availability articles and the Thales HA administration documentation.
- If you require help with key cloning, partition policies, or zeroizing and recovery steps, open a support ticket with Microsoft and/or Thales.

## Frequently asked questions

**Can I change the ERGW public IP SKU on the same VNet without re-deploying HSMs?**
No. Because Dedicated HSMs are deployed into a delegated subnet and ERGWs are bound to the VNet, a simple in-place SKU swap is not a supported migration. Provisioning new VNet/ERGW/HSM resources is the recommended approach.

**Can I export my keys from a Thales Luna HSM and import them into the new HSM?**
Possibly, but only if partition policies permit key export. Most Dedicated HSM deployments have key export disabled for security reasons. If export is disabled, you must recreate keys in the new HSM. See [Migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM](migration-guide.md) for details.

## Next steps

- Read the networking guidance for Dedicated HSM: [Azure Dedicated HSM networking](networking.md).
- Read the high availability guidance: [Azure Dedicated HSM high availability](high-availability.md).
- Use the quickstarts to provision Dedicated HSMs: [Quickstart: Create an Azure Dedicated HSM by using the Azure CLI](quickstart-hsm-azure-cli.md) and [Quickstart: Create an Azure Dedicated HSM with Azure PowerShell](quickstart-create-hsm-powershell.md).
- Review troubleshooting and deprovisioning guidance: [Troubleshooting the Azure Dedicated HSM service](troubleshoot.md).

If you have questions about partition policy, cloning, or a specific migration scenario, engage Thales support and raise a support request through the Azure portal.