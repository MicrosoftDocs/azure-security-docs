---
title: Backup and Restore in Azure Cloud HSM
description: Learn how to back up and restore your Azure Cloud HSM resources, including prerequisites, configuration steps, and validation procedures.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin

# Customer intent: As a security administrator, I need to back up and restore Azure Cloud HSM resources to ensure business continuity and facilitate disaster recovery.

---

# Tutorial: Backup and restore

Microsoft Azure Cloud HSM lets you back up and restore your Cloud HSM, preserving all keys, versions, attributes, tags, and role assignments.

> [!IMPORTANT]
> When you create an Azure Cloud HSM backup, it is protected by a key derived within the HSM. Microsoft doesn't have visibility or access to the derived key protecting your backups. Azure Cloud HSM doesn't support restoring a backup to its source HSM or any Cloud HSM that is already activated. To restore, use another nonactivated Cloud HSM in any preferred region. Otherwise, the restore operation fails, rendering the destination Cloud HSM nonfunctional. 

## Prerequisites

The following prerequisites support Azure Cloud HSM backup and restore operations.  
- Managed service identity configured for Azure Cloud HSM source and destination.
- Azure Blob Storage with RBAC ([Storage Blob Data Contributor](https://github.com/OpenSC/OpenSC)).
- The Azure Cloud HSM destination resource must be in the NotActivated state.

The following configurations aren't supported for backup and restore with Azure Cloud HSM.
- Storage container SAS tokens aren't supported.

## Apply an MSI to Cloud HSM and Create a Storage Account for HSM Backups

### Creating a managed service identity

Create a new user-assigned managed service identity in your existing Azure Cloud HSM resource group. In this guided example, we use CHSM-MSI and CHSM-SERVER-RG, the resource group name referenced in the Azure Cloud HSM onboarding guide example.

```azurepowershell-interactive
# Define parameters for the new managed service identity (MSI)
$identity = @{
    Location          = "<RegionName>"                                         
    ResourceName      = "<MSIName>"                                         
    ResourceGroupName = "<ResourceGroupName>"
    SubscriptionID    = "<SubscriptionID>"     
}

# Create a new user-assigned managed service identity (MSI) in the specified resource group and location
New-AzUserAssignedIdentity -Name $identity.ResourceName -ResourceGroupName $identity.ResourceGroupName -Location $identity.Location
```

### Apply MSI to source and destination Cloud HSM

For Azure Cloud HSM backup and restore operations, a managed service identity (MSI) must be applied to both your source and destination Cloud HSM resources. Each Cloud HSM cluster can have only one MSI. You can either use the same MSI for both the source and destination or use different MSIs for each. In this guided example, we apply the same MSI to both the source and destination Cloud HSM resources.

```azurepowershell-interactive
# Define the source Cloud HSM parameters
$sourceCloudHSM = @{
    Location          = "<RegionName>"                              
    Sku               = @{ "family" = "B"; "Name" = "Standard_B1" } 
    ResourceName      = "<SourceCloudHSMName>"                
    ResourceType      = "microsoft.hardwaresecuritymodules/cloudHsmClusters" 
    ResourceGroupName = "<SourceResourceGroupName>"             
    Force             = $true                                    
}

# Define the destination Cloud HSM parameters
$destinationCloudHSM = @{
    Location          = "<RegionName>"                              
    Sku               = @{ "family" = "B"; "Name" = "Standard_B1" } 
    ResourceName      = "<DestinationCloudHSMName>"            
    ResourceType      = "microsoft.hardwaresecuritymodules/cloudHsmClusters" 
    ResourceGroupName = "<DestinationResourceGroupName>"             
    Force             = $true                                    
}

# Define the Cloud HSM MSI patch payload
$chsmMSIPatch = '{
    "Sku": {
        "Family": "B",
        "Name": "Standard_B1"
    },
    "Location": "<RegionName>",    
    "Identity": {
        "type": "UserAssigned",
        "userAssignedIdentities": {
            "/subscriptions/<SubscriptionID>/resourcegroups/<ResourceGroupName>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<MSIName>": {}
        }
    }
}'

# Construct the source URI
$sourceURI = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($sourceCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($sourceCloudHSM.ResourceName)?api-version=2024-06-30-preview"

# Construct the destination URI
$destinationURI = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($destinationCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($destinationCloudHSM.ResourceName)?api-version=2024-06-30-preview"

# Invoke REST method to update the source Cloud HSM with MSI patch
Invoke-AzRestMethod -Path $sourceURI -Method Put -Payload $chsmMSIPatch

# Invoke REST method to update the destination Cloud HSM with MSI patch
Invoke-AzRestMethod -Path $destinationURI -Method Put -Payload $chsmMSIPatch
```

### Create a storage account in your private VNET for Cloud HSM backups

Set up storage infrastructure for Azure Cloud HSM backup operations by defining and configuring a storage account and associated blob container within a private virtual network. Begin by defining the subscription ID and specifying the storage account parameters, including location, SKU, and type. The process includes creating a new resource group, establishing the storage account with network rules to restrict access to a designated virtual network, and enhancing security through a private endpoint. A private endpoint is set up for secure access, and the "Storage Blob Data Contributor" role is assigned to a specified identity to ensure appropriate permissions for backup tasks. In this example, we create a new resource group named CHSM-BACKUP-RG, a storage account called chsmbackup00, and a blob container named chsmbackupcontainer00, with read/write access granted for both source and destination.

> [!IMPORTANT]
> The minimum RBAC role required is **Storage Blob Data Contributor**. Public storage accounts are accessible over the public internet, so place your storage account behind a private VNET for enhanced security.

```azurepowershell-interactive
# Define the subscription ID
$subscriptionId = "<SubscriptionID>"

# Define storage account parameters
$storageAccount = @{
    Location          = "<RegionName>"                    
    ResourceGroupName = "<BackupResourceGroupName>" 
    AccountName       = "<ResourceName>"      # Name for the storage account
    SkuName           = "<StorageAccountSKU>"     # Storage account SKU (example: Standard_LRS)
    Kind              = "<StorageAccountType>"       #Type of storage account (example: StorageV2)
}

# Define the blob container parameters
$container = @{
    ResourceGroupName  = $storageAccount.ResourceGroupName # Resource group name where the storage account is located
    StorageAccountName = $storageAccount.AccountName      # Name of the storage account
    ContainerName      = "<StorageContainerName>"              # Name for the blob container
}

# Define the private endpoint parameters
# Storage accounts are publicly accessible, its recommended to put it behind a private VNET
$privateEndpoint = @{
    Name              = "<PrivateEndpointName>"
    VnetName          = "<ExistingVNetName>"  # Name of the existing VNet
    SubnetName        = "<ExistingSubnetName>"  # Name of the existing subnet within the VNet
    ResourceGroupName = "<ResourceGroupName>" # Resource group for private VNet and subnet (example: CHSM-CLIENT-RG)
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

# Create the private endpoint for the storage account in the existing VNet
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

# Assign 'Storage Blob Data Contributor' role to the specified identity
New-AzRoleAssignment -RoleDefinitionName $roleAssignment.RoleDefinitionName `
    -PrincipalId $roleAssignment.PrincipalId `
    -Scope $roleAssignment.Scope
```

### Disable shared key access via portal setting

Disable shared key access to enhance security. To do this, navigate to your Azure Storage Account, select Settings >> Configuration, and set 'Allow storage account key access' to 'Disabled'.

## Azure Cloud HSM Backup

### Initiate Cloud HSM backup from source HSM

Start a backup for your source Azure Cloud HSM by sending a POST request with the storage container URI to the backup API endpoint. Monitor the backup's progress by sending a GET request to the URL in the response headers. The script retrieves and shows the backup job status and unique backup ID from the response, confirming the backup has started and tracking its progress.

```azurepowershell-interactive
# Define backup properties, including the URI for the Azure Storage Blob container
$backupProperties = ConvertTo-Json @{
    azureStorageBlobContainerUri = "https://$($container.StorageAccountName).blob.core.windows.net/$($container.ContainerName)"
}

# Construct the URI for the backup operation using the provided parameters
$backupUri = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($sourceCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($sourceCloudHSM.ResourceName)/backup?api-version=2024-06-30-preview"

# Initiate the backup operation by sending a POST request with the backup properties
$response = Invoke-AzRestMethod -Path $backupUri -Method Post -Payload $backupProperties

# Check the backup job status to confirm it succeeded
$jobStatus = Invoke-AzRestMethod -Method 'GET' -Uri $response.Headers.Location
$backupStatus = (ConvertFrom-Json $jobStatus.Content).properties.status

# Extract the backup ID from the job status response
$backupID = (ConvertFrom-Json $jobStatus.Content).properties.backupId

# Output the backup status and backup ID
$backupStatus
$backupID
```

**Expected output**: `$backupStatus` returns "Succeeded", and `$backupID` shows the corresponding backup ID for the backup you initiated.

## Azure Cloud HSM Restore

### Initiate Cloud HSM restore to destination HSM

Start a restore operation for the destination Azure Cloud HSM by providing the necessary restore properties, including the storage container URI and backup ID. The script creates the correct URI for the restore API endpoint and sends a POST request to initiate the restore. To monitor the restore progress, send a GET request to the location specified in the response headers and retrieve the restore job status to confirm its completion.

```azurepowershell-interactive
$restoreProperties = ConvertTo-Json @{
    "azureStorageBlobContainerUri" = "https://$($container.StorageAccountName).blob.core.windows.net/$($container.ContainerName)"
    "backupId" = "$($backupID)"
}

# Set the URI for the restore API endpoint using the subscription ID, resource group, and destination server details
$restoreUri = "/subscriptions/$($identity.SubscriptionID)/resourceGroups/$($destinationCloudHSM.ResourceGroupName)/providers/Microsoft.HardwareSecurityModules/cloudHsmClusters/$($destinationCloudHSM.ResourceName)/restore?api-version=2024-06-30-preview"

# Initiate the restore operation by sending a POST request to the restore API endpoint with the defined properties
$response = Invoke-AzRestMethod -Path $restoreUri -Method Post -Payload $restoreProperties 

# Check the status of the restore job by sending a GET request to the location provided in the response headers
$jobStatus = Invoke-AzRestMethod -Method 'GET' -Uri $response.Headers.Location

# Extract and display the status of the restore job
(ConvertFrom-Json $jobStatus.Content).properties.status
```

**Expected Output**:* $jobStatus should return "Succeeded".

### Validate restore to destination Cloud HSM

After some processing time, the restore operation should indicate "Succeeded," and the Destination Cloud HSM should display an Activation Status of "Active." When you connect to the Destination Cloud HSM where you performed the restore operation, running `azcloudhsm_mgmt_util` and executing `getClusterInfo` should show all three nodes as active and available.

> [!IMPORTANT]
> When connecting to Azure Cloud HSM from the Admin VM, update both the Client and Management configuration files (`azcloudhsm_resource.cfg`) to point to the correct destination Cloud HSM where the restore was performed.

1. Start the `azcloudhsm_mgmt_util` by executing:  

    **Linux**
    ```bash
    cd /usr/local/bin/AzureCloudHSM-ClientSDK-* 
    sudo ./azcloudhsm_mgmt_util ./azcloudhsm_resource.cfg
    ```  
    **Windows**
    ```powershell
    cd azcloudhsm_mgmt_util 
    .\azcloudhsm_mgmt_util.exe .\azcloudhsm_resource.cfg
    ```

2. Inside the management utility, the prompt becomes `cloudmgmt`. To list the cluster information, run: 
    ```bash
    getClusterInfo  
    ```
    **Expected Output:** `getClusterInfo` should confirm that all three nodes are now available for your Azure Cloud HSM cluster.

3. Exit the Management Utility, then launch the Azure Cloud HSM Utility (`azcloudhsm_util`). Sign in as CU and run the `findKey` command.

    > [!IMPORTANT]
    > Key handles can change because they are dynamic; however, key IDs don't change.

    **Linux**
    ```bash
    ./azcloudhsm_util  
    loginHSM -u CU -s cu1 -p user1234
    findKey
    ```  
    **Windows**
    ```powershell
    azcloudhsm_util.exe
    loginHSM -u CU -s cu1 -p user1234  
    findKey
    ```

## Next steps

- [Secure your Azure Cloud HSM](secure-cloud-hsm.md)
- [Network security](network-security.md)
- [Key Management in Azure Cloud HSM](key-management.md)
- [User Management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
