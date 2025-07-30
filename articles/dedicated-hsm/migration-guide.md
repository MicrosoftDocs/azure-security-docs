---
title: Azure Dedicated HSM Migration Guide: Move Keys to Managed or Cloud HSM
description: Learn how to migrate from Azure Dedicated HSM to Azure Managed HSM or Azure Cloud HSM, including key migration restrictions, onboarding steps, and best practices.
author: msmbaldwin
ms.author: mbaldwin
ms.topic: how-to
ms.date: 07/28/2025
services: dedicated-hsm
ms.topic: how-to
ms.service: azure-dedicated-hsm
---

# Azure Dedicated HSM Migration Guide

Azure Dedicated HSM customers interested in the possibility of migrating keys can find guidance here about moving to Azure Managed HSM or Azure Cloud HSM.

> [!WARNING]
> Customers looking to move to Azure Managed HSM or Azure Cloud HSM will not be able to migrate their existing key materials from Azure Dedicated HSM due to known preconditions and restrictions of the Thales Luna HSM. You will need to create new keys in Azure Managed HSM or Azure Cloud HSM when transitioning off Azure Dedicated HSM.

Microsoft does not have access to Dedicated HSMs allocated to customers as devices are allocated within the customers' private IP address space. Because customers have full administrative control of their Dedicated HSMs they are responsible for coordinating with their internal teams to create new keys in Azure Cloud HSM, updating their applications to use new keys in Azure Cloud HSM and testing to ensure safe migration to Azure Cloud HSM.

## Thales Luna HSM restrictions

Exporting HSM protected keys from a Thales Luna HSM into Azure Managed HSM or Azure Cloud HSM is only possible if private key wrapping is on (enabled), and private key cloning is off (disabled). The Thales Luna HSM default setting for partition policy is ‘cloning mode on’. In this mode, private keys are never allowed to exist outside of a trusted Luna HSM in the designated cloning domain.

Customers of Azure Dedicated make use of High-Availability Groups to support their BCDR across multiple devices. Thales requires ‘cloning mode on’ to support that HA feature, concluding that Azure Dedicated HSM customers that have existing keys have cloning mode enabled and thus will not be able to migrate their existing key materials and will have to create new keys in Azure Managed HSM or Azure Cloud HSM.

- Thales Luna HSMs that have partition policies set to ‘cloning mode on’ cannot export keys!

> [!WARNING]
> Changing partition policies on your Thales Luna HSM is a destructive process. If you apply partition change policy to your HSM it will zeroize and all key materials and contents will be lost. If you are not sure of your partition policy state for your Thales Luna HSM you can run [partition showpolicies](https://www.thalesgroup.com/gphsm/luna/7/docs/network/Content/lunacm/commands/partition/partition_showpolicies.htm) on your Azure Dedicated HSM to get a listing of current policies set on your device.

## Getting started with Azure Cloud HSM

Customers can refer to the Azure Cloud HSM onboarding guide, integration guides, and overview documentation to provision and activate their Azure Cloud HSM resources.

### Provision and activate your Azure Cloud HSM

To get started with Azure Cloud HSM, you need to provision and activate your HSM resources. Follow the guides below for detailed instructions.

- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf)

- [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides)

- [Overview of Azure Cloud HSM](/azure/cloud-hsm/overview)

### Create keys in Azure Cloud HSM

Creating keys in Azure Cloud HSM is a straightforward process. You can learn how to create, list, delete, backup, recover, or import keys by referring to the [Azure Cloud Overview](/azure/cloud-hsm/overview), which provides all the necessary commands and guidance for managing HSM keys.

### Read best practices using Azure Cloud HSM

Azure Cloud HSM is a cloud service that safeguards encryption keys. As these keys are sensitive and business critical, make sure to secure access to your Cloud HSMs by allowing only authorized applications and users. Our [Best Practices](/azure/cloud-hsm/secure-cloud-hsm) and article about [Key Management and Security](/azure/cloud-hsm/key-management) and [User Management](/azure/cloud-hsm/user-management) provides an overview of the access model. It explains authentication and authorization, and role-based access control that you will want to adhere to.

## Getting started with Azure Managed HSM

Customers can quickly provision and activate a Managed HSM using the QuickStart references guides shared below:

### Provision and activate a Managed HSM

To begin using Azure Managed HSM, you need to provision and activate it. Use the following guides for step-by-step instructions.

- [Provision and activate a Managed HSM using Azure CLI](/azure/key-vault/managed-hsm/quick-create-cli)

- [Provision and activate a Managed HSM using PowerShell](/azure/key-vault/managed-hsm/quick-create-powershell)

- [Provision and activate a Managed HSM using Azure Resource Manager Template](/azure/key-vault/managed-hsm/quick-create-template)

### Create keys in Azure Managed HSM

Creating keys in Azure Managed HSM is essential for securing your data. Refer to the following resources for detailed instructions.

- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf)

- [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides)

### Read best practices using Azure Managed HSM

Azure Managed HSM is a cloud service that safeguards encryption keys. As these keys are sensitive and business critical, make sure to secure access to your managed HSMs by allowing only authorized applications and users. Our [Best Practices](/azure/key-vault/managed-hsm/best-practices) and article about [Managed HSM Access Control](/azure/key-vault/managed-hsm/access-control) provides an overview of the access model. It explains authentication and authorization, and role-based access control that you will want to adhere to.

## Frequently asked questions

**Can customers migrate Dedicated HSM keys to Managed HSM or Cloud HSM?**

If your Dedicated HSM is enabled with HA Grouping you cannot migrate keys!

- Asymmetric Keys are not migratable from Dedicated HSM to Managed HSM or Cloud HSM when HA Grouping is configured for Luna HSM due to export disabled state.

- Symmetric Keys are migratable regardless of HA Grouping, export enabled/disabled state.

**Can customers change partitions policies on Dedicated HSM to export disabled to migrate keys?**

- No. Changing partition policies for key cloning or key export is a destructive process. Only at the time of partition creation can this policy be set. If you change this policy after creating keys you will factory reset and lose all your key materials. This policy is enforced through Thales firmware.

## Next steps

To learn more about Azure Cloud HSM and Azure Managed HSM, explore the following resources:

- [Azure Cloud HSM Overview](/azure/cloud-hsm/overview): Learn about the features and capabilities of Azure Cloud HSM.
- [Azure Managed HSM Overview](/azure/key-vault/managed-hsm/overview): Understand the benefits and use cases of Azure Managed HSM.
- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf): Step-by-step instructions for getting started with Azure Cloud HSM.
- [Azure Managed HSM Quickstart](/azure/key-vault/managed-hsm/quick-create-cli): Quickstart guide for provisioning and activating Azure Managed HSM.
- [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/OnboardingGuides/Azure%20Cloud%20HSM%20Onboarding.pdf): Step-by-step instructions for getting started with Azure Cloud HSM.
- [Azure Managed HSM Quickstart](/azure/key-vault/managed-hsm/quick-create-cli): Quickstart guide for provisioning and activating Azure Managed HSM.
