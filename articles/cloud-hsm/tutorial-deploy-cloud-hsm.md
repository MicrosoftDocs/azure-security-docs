---
title: Tutorial - Deploying Microsoft Azure Cloud HSM | Microsoft Docs
description: Deploying Azure Cloud HSM into an existing virtual network.
author: keithp
manager: keithp
ms.service: azure-dedicated-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp

#Customer Intent: As an IT Pro, decision maker I am looking for key storage capability within Azure Cloud that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer managed HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offload SSL/TLS processing, certificate authority private key protection, transparent data encryption, including document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only it does not integrate with other Azure services. Cloud HSM does not have a REST API and does not support Encryption at Rest.
---

# Tutorial - Deploying Azure Cloud HSM

For an overview of Azure Cloud HSM, see [What is Azure Cloud HSM?](overview.md)

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin

> [!NOTE]
> Microsoft Azure Cloud HSM is available through Public Preview.  

## Provisioning a Cloud HSM

[comment]: <> (Create an Azure Cloud HSM resource using splatting.)

> [!NOTE]
> Microsoft recommends for Azure Cloud HSM provisioning, initialization, and configuration that customers use the  [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/OnboardingGuides) and [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides).
>
> To perform backup and restore operations in Azure Cloud HSM, it is necessary to create a user-assigned managed identity. This identity is utilized to transfer a Cloud HSM backup to the customer's designated storage account, enabling Business Continuity and Disaster Recovery (BCDR) scenarios. This process includes the provisioning of a new Cloud HSM using an existing backup.

## Deploy Azure Cloud HSM

The following example creates a resource group and your first Cloud HSM instance. When using the example below, update the subscription, resource group, location, and HSM name to a unique resource name. If you specify a resource name for your HSM that already exists in the given region, your deployment fails.

We recommend deploying your Azure Cloud HSM resource in a separate resource group, then your related Client VNET, and VM resources.

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

## Initializing and Configuring a Cloud HSM

[comment]: <> (Cloud HSM customers are directed to Integration Guides and SDK in GitHub.)

For onboarding, deployment, and configuration, use the Azure Cloud HSM guides.
* [Azure Cloud HSM Onboarding Guide](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/OnboardingGuides)
* [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides)

## Deleting a Cloud HSM

Other integration guides, tutorials, and quickstarts in this collection build upon this tutorial. If you plan to continue on to work with subsequent integration guides, tutorials, and quickstarts, you may wish to leave these resources in place.

When no longer needed, you can use the az group delete command to remove the resource group, and all related resources. You can delete the resources as follows:

```azurepowershell-interactive
az group delete --name CHSM-SERVER-RG
```

## Next steps

Integration Guides and [Microsoft Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM) are available to help you facilitate the provisioning and configuration of HSMs into your existing virtual network environment. 

[//]: # (This section to be updated once TOC and Product Links become available)
* [Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM)
* [Key Management in Azure](/azure/security/fundamentals/key-management)
