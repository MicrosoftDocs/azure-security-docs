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

# Migrate Azure Key Vault from access policies to RBAC

Azure Key Vault is undergoing a security change in API version 2026-02-01 in February 2026.

- **Default to Azure role-based access control (RBAC)**: In our investment to protect your Key Vaults from becoming high-impact breach vectors, the new Key Vault API version will set RBAC as the default access configuration, aligning with the Azure portal. Customers currently using access policies are encouraged to migrate to RBAC prior to using the latest version of the API. For more information on why we recommend RBAC, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md). If you decide not to migrate to RBAC, for new Key Vaults you must explicitly set your Access Configuration to use vault access policy before upgrading to API version 2026-02-01.

All earlier versions of the 2026-02-01 API will be retired on February 27, 2027.

We recommend that you follow the steps below and migrate to the new API version once it becomes available in February 2026. For more information about Key Vault access control, see [Secure your key vault](secure-key-vault.md).

> [!WARNING]
> You MUST complete the steps by February 27, 2027 to ensure uninterrupted service.

## Check if your vault's access configuration is set to Azure RBAC or Access Policies

### Check a single vault

# [Azure CLI](#tab/azure-cli)

1. Use the [az keyvault show](/cli/azure/keyvault#az-keyvault-show) command to retrieve vault details:

   ```azurecli
   az keyvault show --name <KeyVaultName> --resource-group <ResourceGroupName>
   ```

2. Check the **Enabled for RBAC Authorization** property for the Key Vault.

# [PowerShell](#tab/azure-powershell)

1. Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to retrieve vault details:

   ```azurepowershell
   Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   ```

2. Check the **Enabled for RBAC Authorization** property for the Key Vault.

---

### Check multiple vaults by Resource Group

# [PowerShell](#tab/azure-powershell)

1. Create the following function in PowerShell:

   ```azurepowershell
   function Get-KeyVaultsFromResourceGroup {
       # Get all Key Vaults in the specified resource group (basic information)
       $keyVaults = Get-AzKeyVault -ResourceGroupName $resourceGroupName
   
       # Iterate through each Key Vault by name and fetch full properties
       foreach ($keyVault in $keyVaults) {
           $keyVaultName = $keyVault.VaultName
   
           # Get the full Key Vault properties
           $keyVaultDetails = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   
           # Access the 'enabledForRbacAuthorization' property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization
   
           # Output Key Vault status based on the 'enabledForRbacAuthorization' property
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled"
           }
       }
   }
   ```

2. Name the ResourceGroup you want to run your function for:

   ```azurepowershell
   $resourceGroupName = "<ResourceGroupName>"
   ```

3. Call function `Get-KeyVaultsFromResourceGroup` to see which vaults in the resource group from step 2 have access policies vs RBAC enabled.

---

### Check multiple vaults by subscription ID

# [PowerShell](#tab/azure-powershell)

1. Create the following function in PowerShell:

   ```azurepowershell
   function Get-KeyVaultsFromSubscription {
       # Get all Key Vaults in the subscription (basic information)
       $keyVaults = Get-AzKeyVault
   
       # Iterate through each Key Vault by name and fetch full properties
       foreach ($keyVault in $keyVaults) {
           $keyVaultName = $keyVault.VaultName
           $resourceGroupName = $keyVault.ResourceGroupName
   
           # Get the full Key Vault properties
           $keyVaultDetails = Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   
           # Access the 'enabledForRbacAuthorization' property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization
   
           # Output Key Vault status based on the 'enabledForRbacAuthorization' property
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled"
           }
       }
   }
   ```

2. Call function `Get-KeyVaultsFromSubscription` to see which vaults in the Subscription have access policies vs RBAC enabled. Note depending on the amount of vaults in your subscription, the function may take more than 10 minutes to run.

---

## For vaults using Azure RBAC

Update all Key Vault ARM, BICEP, Terraform templates and [REST API](/rest/api/keyvault/) calls to use API version 2026-02-01.

## For vaults using Access Policies

Decide if you will migrate to using RBAC-based access (see [Step 4a](#step-4a-migrate-to-azure-rbac-recommended) - Recommended), or if you will continue using access policies (see [Step 4b](#step-4b-continue-using-access-policies)). For more information on access control models, see [Use an Azure RBAC for managing access to Key Vault](rbac-guide.md) and [Azure Key Vault best practices](best-practices.md).

### Migrate to Azure RBAC (recommended)

Use this opportunity to increase your security posture by migrating from vault access policy to Azure RBAC for managing access. For detailed migration guidance, see [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md).

Update all Key Vault ARM, BICEP, Terraform templates and REST API calls to use API version 2026-02-01.

### Continue using Access Policies

If you choose to use Access Policies:

**When using ARM, BICEP, Terraform templates:** 

Update all Key Vault ARM, BICEP, Terraform templates and [REST API](/rest/api/keyvault/) calls to explicitly set `enableRbacAuthorization` to `false` and use API version 2026-02-01.

**When using Create Key Vault commands:**

1. Update your modules:
   - [Update-Module (PowerShellGet) - PowerShell | Microsoft Learn](https://learn.microsoft.com/powershell/module/powershellget/update-module)
   - [Update-AzModule (Az.Tools.Installer) | Microsoft Learn](https://learn.microsoft.com/powershell/module/az.tools.installer/update-azmodule)
   - [How to update the Azure CLI | Microsoft Learn](https://learn.microsoft.com/cli/azure/update-azure-cli)

2. For CLI workflows, use the [az keyvault create](/cli/azure/keyvault#az-keyvault-create) command and set `--enable-rbac-authorization false`:

   ```azurecli
   az keyvault create --name "testCreateTutorial" --resource-group "testResourceGroup" --enable-rbac-authorization false
   ```

3. For PowerShell workflows, use the [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) cmdlet and set `-DisableRbacAuthorization`:

   ```azurepowershell
   New-AzKeyVault -Name "testCreateTutorial" -ResourceGroupName "testResourceGroup" -Location "EastUS" -DisableRbacAuthorization
   ```

**When using Create Resource commands:**

# [Azure CLI](#tab/azure-cli)

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
- [Secure your key vault](best-practices.md)
- [Azure Key Vault REST API reference](/rest/api/keyvault/)

