---
title: Azure Dedicated HSM Migration Guide
description: Guidance on migrating from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM, including restrictions, onboarding steps, and best practices.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 07/28/2025
services: dedicated-hsm
ms.service: azure-dedicated-hsm
---

# Azure Dedicated HSM Migration Guide

Azure Dedicated HSM customers who want to migrate keys can find guidance in this article about moving to [Azure Cloud HSM](../cloud-hsm/overview.md) or [Azure Managed HSM](../key-vault/managed-hsm/overview.md).

> [!WARNING]
> Customers can not migrate existing key materials from Azure Dedicated HSM to Azure Cloud HSM or Azure Managed HSM due to known preconditions and restrictions of the Thales Luna HSM. You must create new keys in Azure Cloud HSM or Azure Managed HSM when transitioning off Azure Dedicated HSM.

Microsoft doesn't have access to Dedicated HSMs allocated to customers because devices are allocated within the customers' private IP address space. Because customers have full administrative control of their Dedicated HSMs, they are responsible for coordinating with their internal teams to create new keys in Azure Cloud HSM, update their applications to use new keys in Azure Cloud HSM, and test to ensure safe migration to Azure Cloud HSM.

## Thales Luna HSM restrictions

Exporting HSM-protected keys from a Thales Luna HSM to Azure Cloud HSM or Azure Managed HSM is possible only if private key wrapping is on (enabled) and private key cloning is off (disabled). The default partition policy setting for Thales Luna HSM is "cloning mode on". In this mode, private keys are never allowed to exist outside of a trusted Luna HSM in the designated cloning domain.

Customers of Azure Dedicated make use of High-Availability (HA) Groups to support their BCDR across multiple devices. Thales requires ‘cloning mode on’ to support the HA feature, concluding that Azure Dedicated HSM customers that have existing keys have cloning mode enabled and thus will not be able to migrate their existing key materials and will have to create new keys in Azure Cloud HSM or Azure Managed HSM.

- Thales Luna HSMs that have partition policies set to ‘cloning mode on’ cannot export keys!

> [!WARNING]
> Changing partition policies on your Thales Luna HSM is a destructive process. If you apply partition change policy to your HSM, it zeroizes and all key materials and contents are lost. If you are not sure of your partition policy state for your Thales Luna HSM, you can run [partition showPolicies](https://thalesdocs.com/gphsm/luna/7/docs/network/Content/lunash/commands/partition/partition_showpolicies.htm) on your Azure Dedicated HSM to get a listing of current policies set on your device.

## Getting started with Azure Cloud HSM

Customers can refer to the Azure Cloud HSM onboarding guide, integration guides, and overview documentation to provision and activate their Azure Cloud HSM resources.

### Provision and activate your Azure Cloud HSM

To get started with Azure Cloud HSM, you need to provision and activate your HSM resources. Follow the guides below for detailed instructions.

- [Overview of Azure Cloud HSM](../cloud-hsm/overview.md)
- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf)
- [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides)

### Create keys in Azure Cloud HSM

Creating keys in Azure cloud HSM is straightforward. You can learn how to create, list, delete, backup, recover, or import keys by referring to the [Azure Cloud HSM Overview](../cloud-hsm/overview.md), which provides all the necessary commands and guidance for managing HSM keys.

### Read best practices using Azure Cloud HSM

Azure Cloud HSM is a cloud service that safeguards encryption keys. As these keys are sensitive and business critical, make sure to secure access to your Cloud HSMs by allowing only authorized applications and users. The [best practices](/azure/cloud-hsm/secure-cloud-hsm) article, along with [key management and security](/azure/cloud-hsm/key-management) and [user management](/azure/cloud-hsm/user-management), provides an overview of the access model. It explains authentication, authorization, and role-based access control that you should follow.

## Getting started with Azure Managed HSM

Customers can quickly provision and activate a Managed HSM using the QuickStart reference guides below:

### Provision and activate a Managed HSM

To begin using Azure Managed HSM, you need to provision and activate it. Use the following guides for step-by-step instructions.

- [Overview of Azure Managed HSM](../key-vault/managed-hsm/overview.md)
- [Provision and activate a Managed HSM using Azure CLI](../key-vault/managed-hsm/quick-create-cli.md)
- [Provision and activate a Managed HSM using PowerShell](../key-vault/managed-hsm/quick-create-powershell.md)
- [Provision and activate a Managed HSM using Azure Resource Manager Template](../key-vault/managed-hsm/quick-create-template.md)

### Create keys in Azure Managed HSM

Creating keys in Azure Managed HSM is essential for securing your data. Refer to the following resources for detailed instructions.

- [Manage keys in an Azure Managed HSM using the Azure CLI](../key-vault/managed-hsm/key-management.md)

### Read best practices using Azure Managed HSM

Azure Managed HSM is a cloud service that safeguards encryption keys. As these keys are sensitive and business critical, make sure to secure access to your managed HSMs by allowing only authorized applications and users. Our [Best Practices](/azure/key-vault/managed-hsm/best-practices) and article about [Managed HSM Access Control](/azure/key-vault/managed-hsm/access-control) provides an overview of the access model. It explains authentication and authorization, and role-based access control that you will want to adhere to.

## Frequently asked questions

**Can customers migrate Dedicated HSM keys to Cloud HSM or Managed HSM?**

If your Dedicated HSM is enabled with HA Grouping, you cannot migrate keys.

- Asymmetric Keys are not migratable from Dedicated HSM to Cloud HSM or Managed HSM when HA Grouping is configured for Luna HSM due to export disabled state.

- Symmetric Keys are migratable regardless of HA Grouping, export enabled/disabled state.

**Can customers change partitions policies on Dedicated HSM to export disabled to migrate keys?**

- No. Changing partition policies for key cloning or key export is a destructive process. Only at the time of partition creation can this policy be set. If you change this policy after creating keys, you will factory reset and lose all your key materials. This policy is enforced through Thales firmware.

## Next steps

To learn more about Azure Cloud HSM and Azure Managed HSM, explore the following resources:

- [Azure Cloud HSM Overview](../cloud-hsm/overview.md): Learn about the features and capabilities of Azure Cloud HSM.
- [Azure Managed HSM Overview](../key-vault/managed-hsm/overview.md): Understand the benefits and use cases of Azure Managed HSM.
- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf): Step-by-step instructions for getting started with Azure Cloud HSM.
- [Azure Managed HSM Quickstart](../key-vault/managed-hsm/quick-create-cli.md): Quickstart guide for provisioning and activating Azure Managed HSM.
