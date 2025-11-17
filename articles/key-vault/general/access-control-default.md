---
title: Migrate Azure Key Vault from access policies to RBAC
description: Learn how to migrate Azure Key Vault access control from access policies to Azure role-based access control (RBAC) to improve security and management.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 11/17/2025
ms.custom: devx-track-azurepowershell, devx-track-azurecli, sfi-image-nochange

#customer intent: As an Azure Key Vault administrator, I want to migrate from access policies to RBAC so that I can improve security and simplify access management.

---

# RBAC and access policies: choosing a default authorization model

Azure Key Vault supports two access control models: access policies and Azure role-based access control (RBAC). Transitioning to RBAC enhances security through granular permissions, centralized access management, and seamless integration with Azure governance tools.

This article guides you through checking your current Key Vault access configuration, migrating from access policies to RBAC, and updating your templates and scripts to the latest API version.

> [!TIP]
> For a complete migration guide with detailed steps—including inventorying access policies, creating RBAC role assignments, enabling RBAC, validating access, and configuring monitoring—see [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md).

## Prerequisites

- At least Contributor access to the Azure subscription or resource group containing your Key Vaults
- Azure CLI or Azure PowerShell installed and configured
- For PowerShell users: Az.KeyVault module (use `Update-Module` to get the latest version)
- For CLI users: Azure CLI version 2.25.0 or later

## Check your Key Vault access configuration

Before migrating, determine whether your Key Vaults use access policies or RBAC. You can check a single vault, all vaults in a resource group, or all vaults in a subscription.

### Check a single Key Vault

# [Azure CLI](#tab/azure-cli)

1. Use the [az keyvault show](/cli/azure/keyvault#az-keyvault-show) command to retrieve the vault properties:

   ```azurecli
   az keyvault show --name <KeyVaultName> --resource-group <ResourceGroupName>
   ```

1. Check the `enabledForRbacAuthorization` property in the output. If it's `true`, RBAC is enabled. If it's `false`, access policies are enabled.

# [Azure PowerShell](#tab/azure-powershell)

1. Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to retrieve the vault properties:

   ```azurepowershell
   Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   ```

1. Check the `EnableRbacAuthorization` property in the output. If it's `True`, RBAC is enabled. If it's `False`, access policies are enabled.

---

### Check multiple Key Vaults in a resource group

# [Azure CLI](#tab/azure-cli)

Use the [az keyvault list](/cli/azure/keyvault#az-keyvault-list) and [az keyvault show](/cli/azure/keyvault#az-keyvault-show) commands to check all Key Vaults in a resource group:

```azurecli
# Set your resource group name
resourceGroupName="<ResourceGroupName>"

# Get all Key Vaults in the resource group and iterate through them
az keyvault list --resource-group $resourceGroupName --query "[].name" -o tsv | while read vaultName; do
    # Get the enabledForRbacAuthorization property
    rbacEnabled=$(az keyvault show --name $vaultName --resource-group $resourceGroupName --query properties.enableRbacAuthorization -o tsv)
    
    # Output Key Vault status
    if [ "$rbacEnabled" = "true" ]; then
        echo "$vaultName: RBAC is enabled"
    else
        echo "$vaultName: Access Policies are enabled"
    fi
done
```

# [Azure PowerShell](#tab/azure-powershell)

Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to check all Key Vaults in a resource group:

1. Create the function:

   ```azurepowershell
   function Get-KeyVaultsFromResourceGroup {
       param(
           [Parameter(Mandatory=$true)]
           [string]$resourceGroupName
       )
       
       # Get all Key Vaults in the specified resource group
       $keyVaults = Get-AzKeyVault -ResourceGroupName $resourceGroupName
   
       # Iterate through each Key Vault and fetch full properties
       foreach ($keyVault in $keyVaults) {
           $keyVaultName = $keyVault.VaultName
   
           # Get the full Key Vault properties
           $keyVaultDetails = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   
           # Check the enabledForRbacAuthorization property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization
   
           # Output Key Vault status
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled"
           }
       }
   }
   ```

1. Run the function:

   ```azurepowershell
   Get-KeyVaultsFromResourceGroup -resourceGroupName "<ResourceGroupName>"
   ```

---

### Check multiple Key Vaults in a subscription

# [Azure CLI](#tab/azure-cli)

Use the [az keyvault list](/cli/azure/keyvault#az-keyvault-list) and [az keyvault show](/cli/azure/keyvault#az-keyvault-show) commands to check all Key Vaults in your subscription:

```azurecli
# Get all Key Vaults in the subscription and iterate through them
az keyvault list --query "[].{name:name, resourceGroup:resourceGroup}" -o tsv | while read name resourceGroup; do
    # Get the enabledForRbacAuthorization property
    rbacEnabled=$(az keyvault show --name $name --resource-group $resourceGroup --query properties.enableRbacAuthorization -o tsv)
    
    # Output Key Vault status
    if [ "$rbacEnabled" = "true" ]; then
        echo "$name: RBAC is enabled"
    else
        echo "$name: Access Policies are enabled"
    fi
done
```

> [!NOTE]
> Depending on the number of vaults in your subscription, this script may take more than 10 minutes to run.

# [Azure PowerShell](#tab/azure-powershell)

Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to check all Key Vaults in your subscription:

1. Create the function:

   ```azurepowershell
   function Get-KeyVaultsFromSubscription {
       # Get all Key Vaults in the subscription
       $keyVaults = Get-AzKeyVault
   
       # Iterate through each Key Vault and fetch full properties
       foreach ($keyVault in $keyVaults) {
           $keyVaultName = $keyVault.VaultName
           $resourceGroupName = $keyVault.ResourceGroupName
   
           # Get the full Key Vault properties
           $keyVaultDetails = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   
           # Check the enabledForRbacAuthorization property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization
   
           # Output Key Vault status
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled"
           }
       }
   }
   ```

1. Run the function:

   ```azurepowershell
   Get-KeyVaultsFromSubscription
   ```

   > [!NOTE]
   > Depending on the number of vaults in your subscription, this function may take more than 10 minutes to run.

---

## Update templates for vaults using RBAC

If your Key Vaults already use RBAC, update all infrastructure-as-code templates and REST API calls to use API version 2026-02-01. This includes:

- Azure Resource Manager (ARM) templates
- Bicep templates
- Terraform templates
- REST API calls

Refer to the [Azure Key Vault REST API documentation](/rest/api/keyvault/) for the latest schema and properties.

## Decide your migration path for vaults using access policies

For Key Vaults currently using access policies, choose one of the following options:

- **Option A (Recommended)**: Migrate to Azure RBAC for improved security and centralized management
- **Option B**: Continue using access policies with updated configuration

For more information on the differences between these authorization systems, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md).

## Migrate to Azure RBAC

Migrating to RBAC is the recommended approach. It improves your security posture by providing granular permissions and centralized access management.

For detailed migration steps including role assignment examples, validation procedures, and troubleshooting guidance, see [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md).

### Map access policies to RBAC roles

1. Review existing access policies and map them to appropriate RBAC roles. Common Key Vault roles include:
   - Key Vault Administrator
   - Key Vault Secrets User
   - Key Vault Certificates Officer
   - Key Vault Crypto Officer

   For a complete mapping of access policy templates to Azure roles, see the [Access policy templates to Azure roles mapping](rbac-migration.md#access-policy-templates-to-azure-roles-mapping) table.

1. Assign RBAC roles to users, groups, or service principals that need access to your Key Vault. For detailed examples, see [Create equivalent RBAC role assignments](rbac-migration.md#create-equivalent-rbac-role-assignments).

1. Update all Key Vault ARM, Bicep, Terraform templates and REST API calls to use API version 2026-02-01 and set `enableRbacAuthorization` to `true`.

1. Test the new RBAC configuration to ensure all users and applications can access resources as expected.

For information on providing access to Key Vault using RBAC, see [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](rbac-guide.md).

## Continue using access policies

If you choose to continue using access policies instead of migrating to RBAC, update your configuration to explicitly set `enableRbacAuthorization` to `false` and use API version 2026-02-01.

### Update templates

For ARM, Bicep, and Terraform templates:

- Set `enableRbacAuthorization` to `false`
- Use API version 2026-02-01
- Include your existing access policies in the template

### Update Key Vault creation commands

Before creating new Key Vaults, ensure you have the latest version of your tools:

- **PowerShell**: Run `Update-Module` or `Update-AzModule`
- **Azure CLI**: Follow the [Azure CLI update guide](/cli/azure/update-azure-cli)

# [Azure CLI](#tab/azure-cli)

Use the [az keyvault create](/cli/azure/keyvault#az-keyvault-create) command with the `--enable-rbac-authorization` parameter set to `false`:

```azurecli
az keyvault create --name "testCreateTutorial" --resource-group "testResourceGroup" --enable-rbac-authorization false
```

# [Azure PowerShell](#tab/azure-powershell)

Use the [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) cmdlet with the `-DisableRbacAuthorization` parameter:

```azurepowershell
New-AzKeyVault -Name "testCreateTutorial" -ResourceGroupName "testResourceGroup" -Location "EastUS" -DisableRbacAuthorization
```

---

### Update resource creation commands

If you use generic resource creation commands, explicitly set the `enableRbacAuthorization` property and API version.

# [Azure CLI](#tab/azure-cli)

Use the [az resource create](/cli/azure/resource#az-resource-create) command to create a Key Vault with access policies:

```azurecli
az resource create \
    --resource-group $resourceGroup \
    --name $vaultName \
    --resource-type "Microsoft.KeyVault/vaults" \
    --location $location \
    --api-version "2026-02-01" \
    --properties '{
        "sku": { "family": "A", "name": "standard" },
        "tenantId": "'$tenantID'",
        "enableRbacAuthorization": false,
        "accessPolicies": []
    }'
```

# [Azure PowerShell](#tab/azure-powershell)

Use the [New-AzResource](/powershell/module/az.resources/new-azresource) cmdlet to create a Key Vault with access policies:

```azurepowershell
New-AzResource `
    -ResourceGroupName $resourceGroupName `
    -ResourceType "Microsoft.KeyVault/vaults" `
    -ResourceName $keyVaultName `
    -Location $location `
    -ApiVersion "2026-02-01" `
    -Properties @{
        sku = @{ family = "A"; name = "standard" }
        tenantId = $TenantId
        enableRbacAuthorization = $false
        accessPolicies = @()
    }
```

---

## Related content

- [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md)
- [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](rbac-guide.md)
- [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md)
- [Azure Key Vault security overview](secure-key-vault.md)
- [Assign Azure roles using Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell)
- [Assign Azure roles using Azure CLI](/azure/role-based-access-control/role-assignments-cli)
