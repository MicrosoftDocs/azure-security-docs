---
title: Migrate Azure Key Vault from access policies to Azure RBAC
description: Learn how to migrate Azure Key Vault access control from access policies to Azure role-based access control (Azure RBAC) to improve security and management.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 11/17/2025
ms.custom: devx-track-azurepowershell, devx-track-azurecli, sfi-image-nochange

#customer intent: As an Azure Key Vault administrator, I want to migrate from access policies to Azure RBAC so that I can improve security and simplify access management.

---

# Prepare for Key Vault API version 2026-02-01: Azure RBAC as default access control

> [!WARNING]
> Starting February 2026, Azure Key Vault API version 2026-02-01 establishes Azure RBAC as the default access control model. All API versions before 2026-02-01 retire on February 27, 2027. 
> 
> This change breaks compatibility. Before February 27, 2027, you **must either**:
>
> - **Azure RBAC (recommended)**: [Set new and existing vaults to Azure RBAC](#step-4-migrate-to-azure-rbac-recommended) and adopt API version 2026-02-01.
> - **Access policies (legacy)**: [Set new vaults to use access policies](#step-5-continue-using-access-policies) and adopt API version 2026-02-01.
>
> **Azure Cloud Shell users**: Cloud Shell always uses the latest API version. If you use Cloud Shell for Key Vault operations, you're already using API version 2026-02-01 and should follow the steps in this article immediately.

Azure Key Vault is implementing an important security enhancement in API version 2026-02-01, releasing in February 2026. To help protect your key vaults and reduce security risks, **the new Key Vault API version establishes Azure RBAC as the default access control model**, consistent with the Azure portal experience.

The ramifications of this change are as follows:

- **New key vaults created with API version 2026-02-01**: All key vaults created with API version 2026-02-01 default to Azure RBAC (`enableRbacAuthorization` = `true`) unless you set `enableRbacAuthorization` to `false` during creation to use access policies.
 
- **Existing key vaults**: Existing key vaults continue using their current access control model. You can migrate vaults between access control models by changing the `enableRbacAuthorization` property.

> [!IMPORTANT]
> To change the `enableRbacAuthorization` property for a key vault, you must have the `Microsoft.Authorization/roleAssignments/write` permission. This permission is included in roles such as Owner and User Access Administrator. For more information, see [Enable Azure RBAC permissions on Key Vault](rbac-guide.md#enable-azure-rbac-permissions-on-key-vault).
 
Migrate key vaults that currently use access policies to Azure RBAC for improved security. For more information on why Azure RBAC is recommended, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md).

Follow the steps in this article to check your current configuration and then use either Azure RBAC (recommended) or access policies (legacy).

## Step 1: Check current configurations

Check if your vault's access configuration is set to Azure RBAC or access policies. Check this configuration through the Azure CLI or PowerShell commands.

After checking your configuration:
- If your vaults use **Azure RBAC** (`enableRbacAuthorization` = `true`), go to [Step 2: For vaults already using Azure RBAC](#step-2-for-vaults-already-using-azure-rbac).
- If your vaults use **access policies** (`enableRbacAuthorization` = `false`), go to [Step 3: For vaults using access policies](#step-3-for-vaults-using-access-policies).

### Check a single vault

# [Azure CLI](#tab-azure-cli)

1. Use the [az keyvault show](/cli/azure/keyvault#az-keyvault-show) command to retrieve vault details:

   ```azurecli
   az keyvault show --name <KeyVaultName> --resource-group <ResourceGroupName>
   ```

1. Check the **Enabled for RBAC Authorization** property (`enableRbacAuthorization`) for the key vault.

# [PowerShell](#tab/azure-powershell)

1. Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to get vault details:

   ```azurepowershell
   Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   ```

1. Check the **Enabled for RBAC Authorization** property (`enableRbacAuthorization`) for the key vault.

---

### Check multiple vaults by resource group

# [Azure CLI](#tab-azure-cli)

Use the [az keyvault list](/cli/azure/keyvault#az-keyvault-list) command to list all vaults in a resource group and check their RBAC authorization status:

```azurecli
# List all key vaults in the resource group and check Azure RBAC status
az keyvault list --resource-group <ResourceGroupName> --query "[].{name:name, rbacEnabled:properties.enableRbacAuthorization}" --output table
```

# [PowerShell](#tab/azure-powershell)

1. Create the following function in PowerShell:

   ```azurepowershell
   function Get-KeyVaultsFromResourceGroup {
       # Get all key vaults in the specified resource group (basic information)
       $keyVaults = Get-AzKeyVault -ResourceGroupName $resourceGroupName
   
       # Iterate through each key vault by name and fetch full properties
       foreach ($keyVault in $keyVaults) {
           $keyVaultName = $keyVault.VaultName
   
           # Get the full key vault properties
           $keyVaultDetails = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   
           # Access the 'enabledForRbacAuthorization' property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization
   
           # Output key vault status based on the 'enabledForRbacAuthorization' property
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled"
           }
       }
   }
   ```

1. Name the resource group you want to run your function for:

   ```azurepowershell
   $resourceGroupName = "<ResourceGroupName>"
   ```

1. Call function `Get-KeyVaultsFromResourceGroup` to see which vaults in the resource group from step 2 have access policies vs Azure RBAC enabled.

---

### Check multiple vaults by subscription ID

# [Azure CLI](#tab-azure-cli)

Use the [az keyvault list](/cli/azure/keyvault#az-keyvault-list) command to list all vaults in your subscription and check their RBAC authorization status:

```azurecli
# List all key vaults in the subscription and check Azure RBAC status
az keyvault list --query "[].{name:name, rbacEnabled:properties.enableRbacAuthorization}" --output table
```

# [PowerShell](#tab/azure-powershell)

1. Create the following function in PowerShell:

   ```azurepowershell
   function Get-KeyVaultsFromSubscription {
       # Get all key vaults in the subscription (basic information)
       $keyVaults = Get-AzKeyVault
   
       # Iterate through each key vault by name and fetch full properties
       foreach ($keyVault in $keyVaults) {
           $keyVaultName = $keyVault.VaultName
           $resourceGroupName = $keyVault.ResourceGroupName
   
           # Get the full key vault properties
           $keyVaultDetails = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   
           # Access the 'enabledForRbacAuthorization' property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization
   
           # Output key vault status based on the 'enabledForRbacAuthorization' property
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled"
           }
       }
   }
   ```

1. Call function `Get-KeyVaultsFromSubscription` to see which vaults in the subscription have access policies versus Azure RBAC enabled. Depending on the number of vaults in your subscription, the function might take more than 10 minutes to run.

---

## Step 2: For vaults already using Azure RBAC

If your key vaults already use Azure RBAC as their access control model, update all Key Vault ARM, BICEP, Terraform templates, and [REST API](/rest/api/keyvault/) calls to use API version 2026-02-01.

## Step 3: For vaults using access policies

If your key vaults use access policies (`enableRbacAuthorization` = `false`), decide if you want to migrate to role-based access (recommended) or continue using access policies. For more information on access control models, see [Use Azure RBAC for managing access to Key Vault](rbac-guide.md) and [Azure Key Vault best practices](secure-key-vault.md).

**Choose your path:**
- **Recommended**: Go to [Step 4: Migrate to Azure RBAC](#step-4-migrate-to-azure-rbac-recommended)
- **Legacy**: Go to [Step 5: Continue using access policies](#step-5-continue-using-access-policies)

## Step 4: Migrate to Azure RBAC (recommended)

Use this opportunity to increase your security posture by migrating from vault access policy to Azure RBAC for managing access. For detailed migration guidance, see [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md).

Update all Key Vault ARM, BICEP, Terraform templates, and REST API calls to use API version 2026-02-01.

## Step 5: Continue using access policies

To continue using access policies, follow the instructions in this section. 

Choose one of the following methods based on your scenario:
- [Using ARM, BICEP, Terraform templates](#using-arm-bicep-terraform-templates)
- [Using Create Key Vault commands](#using-create-key-vault-commands)
- [Using Create Resource commands](#using-create-resource-commands)

#### Using ARM, BICEP, Terraform templates

When creating new key vaults by using API version 2026-02-01, set `enableRbacAuthorization` to `false` in all Key Vault ARM, BICEP, Terraform templates, and [REST API](/rest/api/keyvault/) calls to use access policies.

#### Using Create Key Vault commands

When creating new key vaults by using API version 2026-02-01, you must specify access policies configuration to avoid defaulting to Azure RBAC.

Make sure you have the latest version of the Azure CLI or PowerShell modules.

# [Azure CLI](#tab-azure-cli)

Update Azure CLI to the latest version. For more information, see [How to update the Azure CLI](/cli/azure/update-azure-cli).

# [PowerShell](#tab/azure-powershell)

Update your PowerShell modules to the latest version:
- [Update-Module (PowerShellGet)](/powershell/module/powershellget/update-module)
- [Update-AzModule (Az.Tools.Installer)](/powershell/module/az.tools.installer/update-azmodule)

---
Use the appropriate command to create a key vault with access policies:

# [Azure CLI](#tab-azure-cli)

Use the [az keyvault create](/cli/azure/keyvault#az-keyvault-create) command and set `--enable-rbac-authorization false`:

```azurecli
az keyvault create --name "testCreateTutorial" --resource-group "testResourceGroup" --enable-rbac-authorization false
```

# [PowerShell](#tab/azure-powershell)

Use the [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) cmdlet and set `-DisableRbacAuthorization`:

```azurepowershell
New-AzKeyVault -Name "testCreateTutorial" -ResourceGroupName "testResourceGroup" -Location "EastUS" -DisableRbacAuthorization
```

---

#### Using Create Resource commands

When you create new key vaults by using API version 2026-02-01, set `enableRbacAuthorization` to `false` to use access policies. If you don't specify this property, it defaults to `true` (Azure RBAC).

# [Azure CLI](#tab-azure-cli)

Use the [az resource create](/cli/azure/resource#az-resource-create) command and set `"enableRbacAuthorization": false` and `--api-version "2026-02-01"`:

```azurecli
az resource create --resource-group $resourceGroup --name $vaultName --resource-type "Microsoft.KeyVault/vaults" --location $location --api-version "2026-02-01" --properties '{"sku": { "family": "A", "name": "standard" }, "tenantId": $tenantID,"enableRbacAuthorization": false, "accessPolicies": []}'
```

# [PowerShell](#tab/azure-powershell)

Use the [New-AzResource](/powershell/module/az.resources/new-azresource) cmdlet and set `enableRbacAuthorization = $false` and `-ApiVersion "2026-02-01"`:

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
        accessPolicies = @()}
```

---

## Next steps

- [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md)
- [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md)
- [Use an Azure RBAC for managing access to Key Vault](rbac-guide.md)
- [Secure your key vault](secure-key-vault.md)
- [Azure Key Vault REST API reference](/rest/api/keyvault/)

