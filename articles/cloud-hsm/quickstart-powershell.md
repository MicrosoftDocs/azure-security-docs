---
title: Deploy Azure Cloud HSM using Azure PowerShell
description: Learn how to deploy Azure Cloud HSM into an existing virtual network using Azure PowerShell.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: quickstart
ms.date: 03/20/2025
ms.author: keithp

#Customer Intent: As an IT pro decision maker, I'm looking for key storage capability within the Azure cloud platform that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer-managed HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offload SSL/TLS processing, certificate authority private key protection, and transparent data encryption, including document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only. It doesn't integrate with other Azure services. Cloud HSM doesn't have a REST API and doesn't support encryption at rest.
---

# Deploy Azure Cloud HSM using Azure PowerShell

[Azure Cloud HSM](overview.md) is a highly available, FIPS 140-3 Level 3 validated single-tenant HSM service that enables you to deploy hardware security modules (HSMs) using various methods including Azure CLI, PowerShell, ARM templates, Terraform, or the Azure portal. This quickstart guides you through the deployment process using Azure PowerShell.

## Prerequisites

Before deploying Azure Cloud HSM:

- An Azure account with an active subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- The latest version of [Azure PowerShell](/powershell/azure/install-azure-powershell) installed.
- Appropriate permissions to create resources in your subscription, including HSM resources.
- For production environments, an existing virtual network (VNet) and subnet for configuring [private endpoints](/azure/private-link/private-endpoint-overview).

## Create an Azure Cloud HSM

The following example code creates a resource group and a Cloud HSM instance. You need to update the subscription, resource group, location, and hardware security module (HSM) names to match your environment.

> [!IMPORTANT]
> The HSM name must be unique. If you specify an HSM resource name that already exists in the chosen region, your deployment will fail.

```azurepowershell-interactive
# Define variables for your Cloud HSM deployment
$server = @{
    Location = "<RegionName>"
    Sku = @{"family" = "B"; "Name" = "Standard_B1" }
    ResourceName = "<HSMName>"
    ResourceType = "microsoft.hardwaresecuritymodules/cloudHsmClusters"
    ResourceGroupName = "<ResourceGroupName>"
    Force = $true
}

# Create HSM Cluster Resource Group
New-AzResourceGroup -Name $server.ResourceGroupName -Location $server.Location -Force

# Create HSM Cluster
New-AzResource @server -AsJob -Verbose
```

> [!NOTE]
> We recommend deploying your Cloud HSM resources in a separate resource group from your related client virtual network and virtual machine resources for better management and security isolation.

## Configure managed identity (optional)

For backup and restore operations in Azure Cloud HSM, you need to create a user-assigned managed identity. This identity is used for transferring Cloud HSM backups to your designated storage account, enabling business continuity and disaster recovery (BCDR) scenarios.

If you plan to use backup and restore functionality, you can create and configure a managed identity using the following PowerShell commands:

```azurepowershell-interactive
# Define parameters for the new managed identity
$identity = @{
    Location          = "<RegionName>"                                         
    ResourceName      = "<ManagedIdentityName>"                                         
    ResourceGroupName = "<ResourceGroupName>"
}

# Create a new user-assigned managed identity
New-AzUserAssignedIdentity -Name $identity.ResourceName -ResourceGroupName $identity.ResourceGroupName -Location $identity.Location

# Get subscription ID
$subscriptionId = (Get-AzContext).Subscription.Id

# Define the Cloud HSM managed identity patch payload
$chsmMSIPatch = @{
    Sku = @{
        Family = "B"
        Name = "Standard_B1"
    }
    Location = $server.Location
    Identity = @{
        type = "UserAssigned"
        userAssignedIdentities = @{
            "/subscriptions/$subscriptionId/resourcegroups/$($identity.ResourceGroupName)/providers/Microsoft.ManagedIdentity/userAssignedIdentities/$($identity.ResourceName)" = @{}
        }
    }
} | ConvertTo-Json -Depth 4

# Construct the URI for the Cloud HSM resource
$resourceURI = "/subscriptions/$subscriptionId/resourceGroups/$($server.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($server.ResourceName)?api-version=2024-06-30-preview"

# Update the Cloud HSM resource with the managed identity
Invoke-AzRestMethod -Path $resourceURI -Method Put -Payload $chsmMSIPatch
```

For detailed instructions on configuring backup and restore operations, see [Back up and restore Azure Cloud HSM resources](backup-restore.md).

## Set up networking (recommended)

For production environments, it's strongly recommended to configure a private endpoint for your Cloud HSM to ensure secure communication. The following PowerShell commands can be used to create a private endpoint:

```azurepowershell-interactive
# Define private endpoint parameters
$privateEndpoint = @{
    Name = "<PrivateEndpointName>"
    ResourceGroupName = $server.ResourceGroupName
    Location = $server.Location
    Subnet = $subnet # You need to have $subnet defined with your subnet configuration
    PrivateLinkServiceConnection = @{
        Name = "$($server.ResourceName)-connection"
        PrivateLinkServiceId = "/subscriptions/$subscriptionId/resourceGroups/$($server.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($server.ResourceName)"
        GroupId = "cloudhsmclusters"
    }
}

# Create the private endpoint
New-AzPrivateEndpoint @privateEndpoint
```

> [!TIP]
> Private endpoints are crucial for security as they enable secure connections to your Azure Cloud HSM through a private link, ensuring traffic between your virtual network and the service traverses the Microsoft backbone network. This eliminates exposure to the public internet, as described in [Network security](network-security.md).

## Initialize and configure your HSM

After deploying your Cloud HSM resource through Azure PowerShell, you need to initialize and configure it. This process cannot be done through PowerShell directly and requires the Azure Cloud HSM SDK and Client Tools.

1. **Download the Azure Cloud HSM SDK** from [GitHub](https://github.com/microsoft/MicrosoftAzureCloudHSM/releases).
2. **Install the SDK** on a VM that has network connectivity to your HSM.
3. **Follow the initialization steps** in the [Azure Cloud HSM Onboarding Guide](onboarding-guide.md).

## Clean up resources

If you created a resource group solely for this quickstart and don't need to keep these resources, you can delete the entire resource group:

```azurepowershell-interactive
Remove-AzResourceGroup -Name $server.ResourceGroupName -Force
```

## Troubleshooting common deployment issues

If you encounter issues during deployment:

- **Resource name conflicts**: Ensure your HSM name is unique in the region. If deployment fails with a naming conflict, try a different name.
- **Network connectivity**: If using private endpoints, verify your VM has proper network access to the HSM. See [Network security](network-security.md) for best practices.
- **Authentication failures**: When initializing the HSM, ensure you're using the correct credentials format as detailed in [Authentication in Azure Cloud HSM](authentication.md).
- **Managed identity issues**: If backup operations fail, verify that the managed identity has been properly assigned and has the necessary permissions.

## Next steps

- [Azure Cloud HSM Onboarding Guide](onboarding-guide.md)
- [Configure backup and restore for Cloud HSM](backup-restore.md)
- [Secure your Cloud HSM](secure-cloud-hsm.md)
- [Network security for Cloud HSM](network-security.md)
- [Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM)
- [Key management in Azure](/azure/security/fundamentals/key-management)
