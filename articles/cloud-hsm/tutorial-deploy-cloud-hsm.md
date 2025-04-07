---
title: Deploy Azure Cloud HSM
description: Learn how to deploy Azure Cloud HSM into an existing virtual network.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp

#Customer Intent: As an IT pro decision maker, I'm looking for key storage capability within the Azure cloud platform that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer-managed HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offload SSL/TLS processing, certificate authority private key protection, and transparent data encryption, including document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only. It doesn't integrate with other Azure services. Cloud HSM doesn't have a REST API and doesn't support encryption at rest.
---

# Tutorial: Deploy Azure Cloud HSM

This tutorial walks you through the process of deploying Azure Cloud HSM Preview into a virtual network. For an overview of the service, see [What is Azure Cloud HSM?](overview.md).

In this tutorial, you:

> [!div class="checklist"]
>
> * Provision, initialize, and configure a Cloud HSM instance.
> * Create and deploy a resource group and a Cloud HSM instance.
> * Delete resources that you no longer need.

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Prepare Cloud HSM

For Cloud HSM provisioning, initialization, and configuration, use these guides:

* [Azure Cloud HSM onboarding guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/OnboardingGuides)
* [Azure Cloud HSM integration guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides)

> [!NOTE]
> To perform backup and restore operations in Azure Cloud HSM, you need to create a user-assigned managed identity. This identity is used for transferring a Cloud HSM backup to your designated storage account, to enable business continuity and disaster recovery (BCDR) scenarios. This process includes the provisioning of a new Cloud HSM instance by using an existing backup.

## Deploy Cloud HSM

The following example code creates a resource group and a Cloud HSM instance. When you're using the example code, update the subscription, resource group, location, and hardware security module (HSM) name to a unique resource name. If you specify an HSM resource name that already exists in the chosen region, your deployment fails.

We recommend deploying your Cloud HSM resources in a separate resource group from your related client virtual network and virtual machine resources.

```azurepowershell-interactive
$server = @{
    Location = "<RegionName>"
    Sku = @{"family" = "B"; "Name" = "Standard_B1" }
    ResourceName = "<HSMName>"
    ResourceType = "microsoft.hardwaresecuritymodules/cloudHsmClusters"
    ResourceGroupName = "<ResourceGroupName>"
    Force = $true
}

#Create HSM Cluster Resource Group
New-AzResourceGroup -Name $server.ResourceGroupName -Location $server.Location -Force

#Create HSM Cluster
New-AzResource @server -AsJob -Verbose
```

## Delete resources

Other integration guides, tutorials, and quickstarts for Cloud HSM build on this tutorial. If you plan to work with that documentation, you might want to keep the resources that you created in this tutorial.

When you no longer need the resource group and all related resources, you can use the `az group delete command` to remove them:

```azurepowershell-interactive
az group delete --name CHSM-SERVER-RG
```

## Related content

For more information about the provisioning and configuration of HSMs into an existing virtual network environment, see:

* [Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM)
* [Key management in Azure](/azure/security/fundamentals/key-management)
