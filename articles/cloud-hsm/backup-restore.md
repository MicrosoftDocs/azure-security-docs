---
title: Backup and Restore Azure Cloud HSM Resources
description: Learn how to back up and restore your Azure Cloud HSM resources, including prerequisites, configuration steps, and validation procedures.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin

# Customer intent: As a security administrator, I need to back up and restore Azure Cloud HSM resources to ensure business continuity and facilitate disaster recovery.

---

# Tutorial: Backup and restore Azure Cloud HSM resources

Azure Cloud HSM Preview lets you back up and restore your hardware security module (HSM) in a way that preserves all keys, versions, attributes, tags, and role assignments.

In this tutorial, you:

> [!div class="checklist"]
>
> - Create a managed identity and a storage account for Cloud HSM resources.
> - Initiate a backup from a source Cloud HSM resource.
> - Initiate and validate a restore to a destination Cloud HSM resource.

> [!IMPORTANT]
> When you create a Cloud HSM backup, a key derived within the HSM helps protect it. Microsoft doesn't have visibility or access to the derived key.

## Prerequisites

- An Azure account with an active subscription. You can [create an account for free](https://azure.microsoft.com/free).
- Azure Blob Storage with role-based access control (RBAC) ([Storage Blob Data Contributor](/azure/role-based-access-control/built-in-roles/storage#storage-blob-data-contributor) role).

### Unsupported configurations

Azure Cloud HSM doesn't support:

- Restoring a backup to its source Cloud HSM resource or any Cloud HSM resource that's already activated. To restore, use another nonactivated Cloud HSM resource in any preferred region. Otherwise, the restore operation fails and renders the destination Cloud HSM resource nonfunctional.
- Backup and restore via shared access signature (SAS) tokens for storage containers.

## Apply a managed identity and create a storage account

Use the code in the following sections to apply a managed identity to Azure Cloud HSM and create a storage account for HSM backups.

### Create a managed identity

Create a new user-assigned managed identity in your existing Azure Cloud HSM resource group. In this tutorial, you can use `CHSM-MSI` as the name of the managed identity and `CHSM-SERVER-RG` as the name of the resource group. `CHSM-SERVER-RG` is the name that the [Azure Cloud HSM onboarding guide](onboarding-guide.md) uses as an example resource group.

```azurepowershell-interactive
# Define parameters for the new managed identity
$identity = @{
    Location          = "<RegionName>"                                         
    ResourceName      = "<ManagedIdentityName>"                                         
    ResourceGroupName = "<ResourceGroupName>"
    SubscriptionID    = "<SubscriptionID>"     
}

# Create a new user-assigned managed identity in the specified resource group and location
New-AzUserAssignedIdentity -Name $identity.ResourceName -ResourceGroupName $identity.ResourceGroupName -Location $identity.Location
```

### Apply the managed identity to Cloud HSM resources

For Azure Cloud HSM backup and restore operations, you must apply a managed identity to both your source and destination Cloud HSM resources.

> [!NOTE]
> The destination Cloud HSM resource must be in the `NotActivated` state.

Each Cloud HSM cluster can have only one managed identity. You can use the same managed identity for both the source and destination, or you can use a different managed identity for each. The example in this tutorial applies the same managed identity to both the source and destination Cloud HSM resources.

```azurepowershell-interactive
# Define the parameters for the source Cloud HSM resource
$sourceCloudHSM = @{
    Location          = "<RegionName>"                              
    Sku               = @{ "family" = "B"; "Name" = "Standard_B1" } 
    ResourceName      = "<SourceCloudHSMName>"                
    ResourceType      = "microsoft.hardwaresecuritymodules/cloudHsmClusters" 
    ResourceGroupName = "<SourceResourceGroupName>"             
    Force             = $true                                    
}

# Define the parameters for the destination Cloud HSM resource
$destinationCloudHSM = @{
    Location          = "<RegionName>"                              
    Sku               = @{ "family" = "B"; "Name" = "Standard_B1" } 
    ResourceName      = "<DestinationCloudHSMName>"            
    ResourceType      = "microsoft.hardwaresecuritymodules/cloudHsmClusters" 
    ResourceGroupName = "<DestinationResourceGroupName>"             
    Force             = $true                                    
}

# Define the Cloud HSM managed identity patch payload
$chsmMSIPatch = '{
    "Sku": {
        "Family": "B",
        "Name": "Standard_B1"
    },
    "Location": "<RegionName>",    
    "Identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<ManagedIdentityName>": {}
        }
    }
}'

# Construct the source URI
$sourceURI = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($sourceCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($sourceCloudHSM.ResourceName)?api-version=2024-06-30-preview"

# Construct the destination URI
$destinationURI = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($destinationCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($destinationCloudHSM.ResourceName)?api-version=2024-06-30-preview"

# Invoke the REST method to update the source Cloud HSM resource with the managed identity patch
Invoke-AzRestMethod -Path $sourceURI -Method Put -Payload $chsmMSIPatch

# Invoke the REST method to update the destination Cloud HSM resource with the managed identity patch
Invoke-AzRestMethod -Path $destinationURI -Method Put -Payload $chsmMSIPatch
```

### Create a storage account in a private virtual network

Set up storage infrastructure for Azure Cloud HSM backup operations by defining and configuring a storage account and an associated blob container within a private virtual network.

Begin by defining the subscription ID. Specify the storage account parameters, including the location, product tier, and type.

The process includes creating a new resource group, establishing the storage account with network rules to restrict access to a designated virtual network, and enhancing security through a private endpoint. The Storage Blob Data Contributor role is assigned to a specified identity to ensure appropriate permissions for backup tasks.

In the following code, you can use the following examples:

- `CHSM-BACKUP-RG` for the name of the resource group
- `chsmbackup00` for the name of the storage account
- `chsmbackupcontainer00` for the name of the blob container

Read/write access is granted for both the source and the destination.

> [!IMPORTANT]
> The minimum RBAC role required is Storage Blob Data Contributor. Public storage accounts are accessible over the public internet, so place your storage account behind a private virtual network for enhanced security.

```azurepowershell-interactive
# Define the subscription ID
$subscriptionId = "<SubscriptionID>"

# Define storage account parameters
$storageAccount = @{
    Location          = "<RegionName>"                    
    ResourceGroupName = "<BackupResourceGroupName>" 
    AccountName       = "<ResourceName>"      # Name of the storage account
    SkuName           = "<StorageAccountSKU>"     # Storage account tier (example: Standard_LRS)
    Kind              = "<StorageAccountType>"       #Type of storage account (example: StorageV2)
}

# Define the blob container parameters
$container = @{
    ResourceGroupName  = $storageAccount.ResourceGroupName # Resource group name where the storage account is located
    StorageAccountName = $storageAccount.AccountName      # Name of the storage account
    ContainerName      = "<StorageContainerName>"              # Name of the blob container
}

# Define the private endpoint parameters
# Storage accounts are publicly accessible, so put it behind a private virtual network
$privateEndpoint = @{
    Name              = "<PrivateEndpointName>"
    VnetName          = "<ExistingVNetName>"  # Name of the existing virtual network
    SubnetName        = "<ExistingSubnetName>"  # Name of the existing subnet within the virtual network
    ResourceGroupName = "<ResourceGroupName>" # Resource group for private virtual network and subnet (example: CHSM-CLIENT-RG)
}

# Define the role assignment parameters
$roleAssignment = @{
    RoleDefinitionName = "Storage Blob Data Contributor"  # Minimum RBAC role required
    PrincipalId        = "<PrincipalId>"  # The ID of the managed identity or user to assign the role to
    Scope              = "/subscriptions/$($subscriptionId)/resourceGroups/$($storageAccount.ResourceGroupName)/providers/Microsoft.Storage/storageAccounts/$($storageAccount.AccountName)"
}

# Create a new resource group with the specified name and location
New-AzResourceGroup -Name $storageAccount.ResourceGroupName -Location $storageAccount.Location -Force

# Create a new storage account in the specified resource group and location
# This command sets up the storage account needed for backup operations
New-AzStorageAccount -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $storageAccount.AccountName `
    -Location $storageAccount.Location `
    -SkuName $storageAccount.SkuName `
    -Kind $storageAccount.Kind

# Retrieve the storage account key
$storageAccountKey = (Get-AzStorageAccountKey -ResourceGroupName $storageAccount.ResourceGroupName -Name $storageAccount.AccountName)[0].Value

# Create the storage context
$storageAccountContext = New-AzStorageContext -StorageAccountName $storageAccount.AccountName -StorageAccountKey $storageAccountKey

# Create the blob container in the storage account
New-AzStorageContainer -Name $container.ContainerName -Context $storageAccountContext

# Retrieve the virtual network and subnet object
$vnet = Get-AzVirtualNetwork -ResourceGroupName $privateEndpoint.ResourceGroupName -Name $privateEndpoint.VnetName
$subnet = $vnet.Subnets | Where-Object { $_.Name -eq $privateEndpoint.SubnetName }

# Create the private endpoint for the storage account in the existing virtual network
New-AzPrivateEndpoint -ResourceGroupName $storageAccount.ResourceGroupName `
    -Name $privateEndpoint.Name `
    -Location $storageAccount.Location `
    -PrivateLinkServiceConnection @(
        @{
            Name = "$($storageAccount.AccountName)-connection"
            PrivateLinkServiceConnectionState = @{
                Status = "Approved"
                Description = "Private Endpoint Connection"
            }
            PrivateLinkServiceId = "/subscriptions/$subscriptionId/resourceGroups/$($storageAccount.ResourceGroupName)/providers/Microsoft.Storage/storageAccounts/$($storageAccount.AccountName)"
            GroupIds = @("blob")  # Add this parameter with the required group ID
        }
    ) `
    -Subnet $subnet 

# Assign the Storage Blob Data Contributor role to the specified identity
New-AzRoleAssignment -RoleDefinitionName $roleAssignment.RoleDefinitionName `
    -PrincipalId $roleAssignment.PrincipalId `
    -Scope $roleAssignment.Scope
```

### Disable shared key access via portal setting

Disable shared key access to enhance security:

1. In the Azure portal, go your Azure storage account.
1. Select **Settings** > **Configuration**.
1. Set **Allow storage account key access** to **Disabled**.

## Initiate a backup from the source Cloud HSM resource

Start a backup for your source Cloud HSM resource by sending a `POST` request with the storage container URI to the backup API endpoint. Monitor the backup's progress by sending a `GET` request to the URL in the response headers.

The following script retrieves and shows the backup job status and the unique backup ID from the response. The status confirms that the backup started and tracks its progress.

```azurepowershell-interactive
# Define backup properties, including the URI for the Azure Blob Storage container
$backupProperties = ConvertTo-Json @{
    azureStorageBlobContainerUri = "https://$($container.StorageAccountName).blob.core.windows.net/$($container.ContainerName)"
}

# Construct the URI for the backup operation by using the provided parameters
$backupUri = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($sourceCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($sourceCloudHSM.ResourceName)/backup?api-version=2024-06-30-preview"

# Initiate the backup operation by sending a POST request with the backup properties
$response = Invoke-AzRestMethod -Path $backupUri -Method Post -Payload $backupProperties

# Check the backup job status to confirm that it succeeded
$jobStatus = Invoke-AzRestMethod -Method 'GET' -Uri $response.Headers.Location
$backupStatus = (ConvertFrom-Json $jobStatus.Content).properties.status

# Extract the backup ID from the job status response
$backupID = (ConvertFrom-Json $jobStatus.Content).properties.backupId

# Output the backup status and backup ID
$backupStatus
$backupID
```

**Expected output**: `$backupStatus` returns `Succeeded`, and `$backupID` shows the corresponding ID for the backup that you initiated.

## Initiate a restore to the destination Cloud HSM resource

Start a restore operation for the destination Cloud HSM resource by providing the necessary restore properties, including the storage container URI and backup ID.

The following script creates the correct URI for the restore API endpoint and sends a `POST` request to initiate the restore. To monitor the restore progress, send a `GET` request to the location specified in the response headers and retrieve the restore job status to confirm its completion.

```azurepowershell-interactive
$restoreProperties = ConvertTo-Json @{
    "azureStorageBlobContainerUri" = "https://$($container.StorageAccountName).blob.core.windows.net/$($container.ContainerName)"
    "backupId" = "$($backupID)"
}

# Set the URI for the restore API endpoint by using the subscription ID, resource group, and destination server details
$restoreUri = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($destinationCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($destinationCloudHSM.ResourceName)/restore?api-version=2024-06-30-preview"

# Initiate the restore operation by sending a POST request to the restore API endpoint with the defined properties
$response = Invoke-AzRestMethod -Path $restoreUri -Method Post -Payload $restoreProperties 

# Check the status of the restore job by sending a GET request to the location provided in the response headers
$jobStatus = Invoke-AzRestMethod -Method 'GET' -Uri $response.Headers.Location

# Extract and display the status of the restore job
(ConvertFrom-Json $jobStatus.Content).properties.status
```

**Expected output**: `$jobStatus` should return `Succeeded`.

### Validate the restore to the destination Cloud HSM resource

After some processing time, the restore operation should indicate `Succeeded`, and the destination Cloud HSM resource should display an activation status of `Active`. When you connect to the destination Cloud HSM resource where you performed the restore operation, running `azcloudhsm_mgmt_util` and `getClusterInfo` should show all three nodes as active and available.

> [!IMPORTANT]
> When you're connecting to Azure Cloud HSM from the administrative virtual machine, update both the client and management configuration files (`azcloudhsm_resource.cfg`) to point to the correct destination Cloud HSM resource where you performed the restore.

1. Start `azcloudhsm_mgmt_util` by running one of the following commands.  

    Linux:

    ```bash
    cd /usr/local/bin/AzureCloudHSM-ClientSDK-* 
    sudo ./azcloudhsm_mgmt_util ./azcloudhsm_resource.cfg
    ```

    Windows:

    ```powershell
    cd azcloudhsm_mgmt_util 
    .\azcloudhsm_mgmt_util.exe .\azcloudhsm_resource.cfg
    ```

2. Inside the management tool, the prompt becomes `cloudmgmt`. To list the cluster information, run:

    ```bash
    getClusterInfo  
    ```

    **Expected output**: `getClusterInfo` should confirm that all three nodes are now available for your Azure Cloud HSM cluster.

3. Close the management tool, and then open the Azure Cloud HSM tool (`azcloudhsm_util`). Sign in as `CU` and run the `findKey` command.

    > [!IMPORTANT]
    > Key handles can change because they're dynamic. However, key IDs don't change.

    Linux:

    ```bash
    ./azcloudhsm_util  
    loginHSM -u CU -s cu1 -p user1234
    findKey
    ```  

    Windows:

    ```powershell
    azcloudhsm_util.exe
    loginHSM -u CU -s cu1 -p user1234  
    findKey
    ```

## Related content

- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Network security for Azure Cloud HSM](network-security.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
