---
title: Deploy Azure Cloud HSM by Using Azure PowerShell
description: Learn how to deploy Azure Cloud HSM into an existing virtual network by using Azure PowerShell.
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: quickstart
ms.date: 03/20/2025
ms.author: keithp

#customer intent: As an IT pro decision-maker, I'm looking for key storage capability within the Azure cloud platform that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer-owned HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offload of SSL/TLS processing, CA private key protection, and transparent data encryption. It also supports document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only. It doesn't integrate with other Azure services, doesn't have a REST API, and doesn't support encryption at rest.
---

# Quickstart: Deploy Azure Cloud HSM by using Azure PowerShell

[Azure Cloud HSM](overview.md) is a highly available, FIPS 140-3 Level 3 validated single-tenant service that enables you to deploy hardware security modules (HSMs) by using various methods. These methods include the Azure CLI, Azure PowerShell, Azure Resource Manager templates (ARM templates), Terraform, or the Azure portal. This quickstart guides you through the deployment process in Azure PowerShell.

## Prerequisites

- An Azure account with an active subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.
- The latest version of [Azure PowerShell](/powershell/azure/install-azure-powershell) installed.
- Appropriate permissions to create resources in your subscription, including HSM resources.
- For production environments, an existing virtual network and subnet for configuring [private endpoints](/azure/private-link/private-endpoint-overview).

## Create an Azure Cloud HSM instance

The following example code creates a resource group and a Cloud HSM instance. You need to update the subscription, resource group, location, and HSM name to match your environment.

> [!IMPORTANT]
> The HSM name must be unique. If you specify an HSM name that already exists in the chosen region, your deployment will fail.

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

# Create an HSM cluster resource group
New-AzResourceGroup -Name $server.ResourceGroupName -Location $server.Location -Force

# Create an HSM cluster
New-AzResource @server -AsJob -Verbose
```

> [!NOTE]
> We recommend that you deploy your Cloud HSM resources in a separate resource group from your related client virtual network and virtual machine (VM) resources. Using a separate resource group provides better management and security isolation.

## Configure managed identity (optional)

For backup and restore operations in Azure Cloud HSM, you need to create a user-assigned managed identity. This identity is used for transferring Cloud HSM backups to your designated storage account in business continuity and disaster recovery (BCDR) scenarios.

If you plan to use backup and restore functionality, you can create and configure a managed identity by using the following Azure PowerShell commands:

```azurepowershell-interactive
# Define parameters for the new managed identity
$identity = @{
    Location          = "<RegionName>"                                         
    ResourceName      = "<ManagedIdentityName>"                                         
    ResourceGroupName = "<ResourceGroupName>"
}

# Create a new user-assigned managed identity
New-AzUserAssignedIdentity -Name $identity.ResourceName -ResourceGroupName $identity.ResourceGroupName -Location $identity.Location

# Get the subscription ID
$subscriptionId = (Get-AzContext).Subscription.Id

# Define the Cloud HSM managed identity's patch payload
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

For production environments, we strongly recommend that you configure a private endpoint for your Cloud HSM instance to help ensure secure communication. You can use the following Azure PowerShell commands to create a private endpoint:

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
> Private endpoints are crucial for security. They enable secure connections to your Azure Cloud HSM instance through a private link. These connections ensure that traffic between your virtual network and the service traverses the Microsoft backbone network. This configuration eliminates exposure to the public internet, as described in [Network security for Azure Cloud HSM](network-security.md).

## Deploy your Cloud HSM resource

When you run the `New-AzResource` command with the `-AsJob` parameter, it creates a background job to deploy your Cloud HSM resource. You can check the status of the deployment by running:

```azurepowershell-interactive
Get-Job -Id <JobId> | Receive-Job
```

In the preceding command, `<JobId>` is the ID that the system returned when you ran the `New-AzResource` command.

The deployment is complete when you see a successful result from the job or when you can verify that the resource exists in your Azure subscription.

## Initialize and configure your HSM

You can't accomplish Azure Cloud HSM activation and configuration through Azure PowerShell directly. You need the Azure Cloud HSM SDK and client tools.

After you deploy your Cloud HSM resource through Azure PowerShell, follow these steps:

1. Download and install the Azure Cloud HSM SDK from [GitHub](https://github.com/microsoft/MicrosoftAzureCloudHSM/releases) on a VM that has network connectivity to your HSM.
2. Initialize and configure your HSM by following the detailed steps in the [Azure Cloud HSM onboarding guide](onboarding-guide.md).
3. Establish user management with appropriate cryptographic officers and users, as described in [User management in Azure Cloud HSM](user-management.md).
4. Implement proper key management practices to help ensure optimal security and performance, as outlined in [Key management in Azure Cloud HSM](key-management.md).

## Clean up resources

If you created a resource group solely for this quickstart and you don't need to keep these resources, you can delete the entire resource group:

```azurepowershell-interactive
Remove-AzResourceGroup -Name $server.ResourceGroupName -Force
```

## Troubleshoot common deployment problems

If you encounter problems during deployment:

- **Resource name conflicts**: Ensure that your HSM name is unique in the region. If deployment fails with a naming conflict, try a different name.
- **Network connectivity problems**: If you're using private endpoints, verify that your VM has proper network access to the HSM. For best practices, see [Network security for Azure Cloud HSM](network-security.md).
- **Authentication failures**: When you're initializing the HSM, ensure that you use the correct format for credentials, as detailed in [Authentication in Azure Cloud HSM](authentication.md).
- **Managed identity problems**: If backup operations fail, verify that the managed identity was properly assigned and has the necessary permissions.

## Related content

- [Azure Cloud HSM onboarding guide](onboarding-guide.md)
- [Back up and restore Azure Cloud HSM resources](backup-restore.md)
- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Network security for Azure Cloud HSM](network-security.md)
- [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md)
