---
title: Transition from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM
description: Guidance on transitioning from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM, including restrictions, onboarding steps, and best practices.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 07/28/2025
services: dedicated-hsm
ms.service: azure-dedicated-hsm
---

# Transition from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM

Azure Dedicated HSM customers who want to transition to [Azure Cloud HSM](../cloud-hsm/overview.md) or [Azure Managed HSM](../key-vault/managed-hsm/overview.md) can find guidance in this article. This transition involves creating new keys and updating applications to use the new services.

> [!WARNING]
> Customers cannot migrate existing key materials from Azure Dedicated HSM to Azure Cloud HSM or Azure Managed HSM due to known restrictions of the Thales Luna HSM. You must create new keys in Azure Cloud HSM or Azure Managed HSM when transitioning off Azure Dedicated HSM.

Microsoft doesn't have access to Dedicated HSMs allocated to customers because devices are allocated within the customers' private IP address space. Customers are responsible for coordinating with their internal teams to create new keys in Azure Cloud HSM or Azure Managed HSM, update their applications to use the new keys, and test to ensure a smooth transition.

## Thales Luna HSM restrictions

Exporting HSM-protected keys from a Thales Luna HSM to Azure Cloud HSM or Azure Managed HSM is not possible if the default partition policy setting, "key export," is disabled. This mode prevents private keys from existing outside of a trusted Luna HSM in the designated cloning domain.

Azure Dedicated HSM customers using High-Availability (HA) Groups to support their BCDR across multiple devices typically have "cloning mode on" enabled and "key export" disabled. As a result, existing key materials cannot be transitioned, and new keys must be created in Azure Cloud HSM or Azure Managed HSM.

Thales Luna HSMs with "cloning mode on" and "key export" disabled cannot export keys.

> [!WARNING]
> Changing partition policies on your Thales Luna HSM is a destructive process. If you apply a partition policy change, it zeroizes the HSM, and all key materials and contents are lost. If you are unsure of your partition policy state, you can run [partition showPolicies](https://thalesdocs.com/gphsm/luna/7/docs/network/Content/lunash/commands/partition/partition_showpolicies.htm) on your Azure Dedicated HSM to view the current policies.

## Transitioning to Azure Cloud HSM

Customers can refer to the Azure Cloud HSM onboarding guide, integration guides, and overview documentation to provision and activate their Azure Cloud HSM resources. For additional details on usage scenarios and best practices, see the [Azure Cloud HSM FAQ](../cloud-hsm/faq.yml#what-usage-scenarios-best-suit-azure-cloud-hsm).

### Provision and activate your Azure Cloud HSM

To get started with Azure Cloud HSM, you need to provision and activate your HSM resources. Follow the guides below for detailed instructions.

- [Overview of Azure Cloud HSM](../cloud-hsm/overview.md)
- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf)
- [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides)

### Create keys in Azure Cloud HSM

Creating keys in Azure Cloud HSM is straightforward. You can learn how to create, list, delete, backup, recover, or import keys by referring to the [Azure Cloud HSM Overview](../cloud-hsm/overview.md), which provides all the necessary commands and guidance for managing HSM keys.

### Read best practices using Azure Cloud HSM

Azure Cloud HSM is a cloud service that safeguards encryption keys. As these keys are sensitive and business critical, make sure to secure access to your Cloud HSMs by allowing only authorized applications and users. The [best practices](/azure/cloud-hsm/secure-cloud-hsm) article, along with [key management and security](/azure/cloud-hsm/key-management) and [user management](/azure/cloud-hsm/user-management), provides an overview of the access model. It explains authentication, authorization, and role-based access control that you should follow.

## Transitioning to Azure Managed HSM

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

No, existing keys cannot be migrated due to restrictions in the Thales Luna HSM. Customers must create new keys in Azure Cloud HSM or Azure Managed HSM.

**Can customers change partition policies on Dedicated HSM to enable key export?**

No. Changing partition policies for key cloning or key export is a destructive process. Only at the time of partition creation can this policy be set. If you change this policy after creating keys, you will factory reset and lose all your key materials. This policy is enforced through Thales firmware.

## Next steps

To learn more about Azure Cloud HSM and Azure Managed HSM, explore the following resources:

- [Azure Cloud HSM Overview](../cloud-hsm/overview.md): Learn about the features and capabilities of Azure Cloud HSM.
- [Azure Managed HSM Overview](../key-vault/managed-hsm/overview.md): Understand the benefits and use cases of Azure Managed HSM.
- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf): Step-by-step instructions for getting started with Azure Cloud HSM.
- [Azure Managed HSM Quickstart](../key-vault/managed-hsm/quick-create-cli.md): Quickstart guide for provisioning and activating Azure Managed HSM.
- [Azure Cloud HSM FAQ](../cloud-hsm/faq.yml): Explore usage scenarios and best practices for Azure Cloud HSM.
