---
title: Prepare for Key Vault API version 2026-02-01 and later - Azure RBAC as default
description: Learn how to prepare for Azure Key Vault API version 2026-02-01 and later, which establishes Azure RBAC as the default access control model and retires earlier API versions by February 2027.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 03/24/2026
ms.custom: devx-track-azurepowershell, devx-track-azurecli, sfi-image-nochange

#customer intent: As an Azure Key Vault administrator, I want to migrate from access policies to Azure RBAC so that I can improve security and simplify access management.

---

# Prepare for Key Vault API version 2026-02-01 and later: Azure RBAC as default access control

Azure Key Vault API version 2026-02-01 and later change the default access control model for new vaults to Azure RBAC, consistent with the Azure portal experience. Both Azure RBAC and access policies remain fully supported. API version 2026-02-01 is available in public Azure regions, Mooncake, and Fairfax.

- **New key vault creation behavior**: When you create a new vault with API version `2026-02-01` or later, the default access control model is Azure RBAC (`enableRbacAuthorization = true`). This default applies only to **create** operations. To use access policies for new vaults, set `enableRbacAuthorization` to `false` at creation time.
- **Existing key vault behavior**: Existing vaults keep their current access control model unless you explicitly change `enableRbacAuthorization`. Using API version `2026-02-01` or later to update a vault does not automatically change access control. Vaults where `enableRbacAuthorization` is `null` (from older API versions) continue using access policies.

> [!IMPORTANT]
> All Key Vault Control Plane API versions before 2026-02-01 retire on February 27, 2027. Adopt API version 2026-02-01 or later before this date. Data Plane APIs are not affected.
>
> Preview API versions (except 2026-04-01-preview) are being deprecated with a 90-day notice period.
> 
> Note that Azure Cloud Shell always uses the latest API version. If you have scripts that run in Cloud Shell, ensure they are compatible with API version 2026-02-01 or later.
>
> Control plane management SDKs supporting API version 2026-02-01 are available for all languages. For package details, see [What's new for Azure Key Vault](whats-new.md#control-plane-sdk-releases).

We encourage you to migrate key vaults that currently use access policies (legacy) to Azure RBAC for improved security. For more information on why Azure RBAC is recommended, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md).

## What you need to do

If you already know your vault's access control model, skip to [Determine your next steps](#determine-your-next-steps). Otherwise, [check your current configuration](#check-your-current-configuration) first.

> [!IMPORTANT]
> To change the `enableRbacAuthorization` property for a key vault, you must have the `Microsoft.Authorization/roleAssignments/write` permission. This permission is included in roles such as Owner and User Access Administrator. For more information, see [Enable Azure RBAC permissions on Key Vault](rbac-guide.md#enable-azure-rbac-permissions-on-key-vault).

## Check your current configuration

Check if your vault's access configuration is set to Azure RBAC or access policies. Check this configuration through the Azure CLI or PowerShell commands.

After checking your configuration:
- If your vaults use **Azure RBAC** (`enableRbacAuthorization` = `true`), go to [Vaults using Azure RBAC](#vaults-using-azure-rbac).
- If your vaults use **access policies (legacy)** (`enableRbacAuthorization` = `false` or `null`), go to [Vaults using access policies](#vaults-using-access-policies).

### Check a single vault

# [Azure CLI](#tab/azure-cli)

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

# [Azure CLI](#tab/azure-cli)

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
   
           # Access the 'EnableRbacAuthorization' property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization

           # Output key vault status based on the 'EnableRbacAuthorization' property
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled (enableRbacAuthorization is false or null)"
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

# [Azure CLI](#tab/azure-cli)

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
   
           # Access the 'EnableRbacAuthorization' property
           $enabledForRbac = $keyVaultDetails.EnableRbacAuthorization

           # Output key vault status based on the 'EnableRbacAuthorization' property
           if ($enabledForRbac -eq $true) {
               Write-Output "${keyVaultName}: RBAC is enabled"
           } else {
               Write-Output "${keyVaultName}: Access Policies are enabled (enableRbacAuthorization is false or null)"
           }
       }
   }
   ```

1. Call function `Get-KeyVaultsFromSubscription` to see which vaults in the subscription have access policies versus Azure RBAC enabled. Depending on the number of vaults in your subscription, the function might take more than 10 minutes to run.

---

<a name="update-your-configuration"></a>

## Determine your next steps

Based on your current access control model, follow the appropriate guidance below.

### Vaults using Azure RBAC

If your key vaults already use Azure RBAC, no access control changes are needed. However, you must update all Key Vault ARM, BICEP, Terraform templates, and [REST API](/rest/api/keyvault/) calls to use API version 2026-02-01 or later before February 27, 2027, when older API versions retire.

### Vaults using access policies

If your key vaults use access policies (legacy) (`enableRbacAuthorization` = `false` or `null`), decide if you want to migrate to role-based access (recommended) or continue using access policies. For more information on access control models, see [Use Azure RBAC for managing access to Key Vault](rbac-guide.md) and [Azure Key Vault best practices](secure-key-vault.md).

Choose your path:

- **Azure RBAC (recommended)**: [Migrate to Azure RBAC](#migrate-to-azure-rbac-recommended) for improved security.
- **Access policies (legacy)**: [Continue using access policies](#continue-using-access-policies) by setting `enableRbacAuthorization` to `false` when creating new vaults.

<a name="step-4-migrate-to-azure-rbac-recommended"></a>

#### Migrate to Azure RBAC (recommended)

Use this opportunity to increase your security posture by migrating from vault access policies to Azure RBAC. For detailed migration guidance, see [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md).

After migrating, update all Key Vault ARM, BICEP, Terraform templates, and REST API calls to use API version 2026-02-01 or later.

<a name="step-5-continue-using-access-policies"></a>

#### Continue using access policies

Access policies remain a fully supported access control model.

- **Existing vaults**: Vaults already using access policies continue to work without changes. Just ensure your ARM, BICEP, Terraform templates, and REST API calls use API version 2026-02-01 or later before February 27, 2027.
- **New vaults**: When creating new vaults with API version 2026-02-01 or later, you must explicitly set `enableRbacAuthorization` to `false` to use access policies, as described below.

Choose one of the following methods based on your scenario:
- [Using ARM, BICEP, Terraform templates](#using-arm-bicep-terraform-templates)
- [Using Create Key Vault commands](#using-create-key-vault-commands)
- [Using Create Resource commands](#using-create-resource-commands)

##### Using ARM, BICEP, Terraform templates

When creating new key vaults by using API version 2026-02-01 or later, set `enableRbacAuthorization` to `false` in all Key Vault ARM, BICEP, Terraform templates, and [REST API](/rest/api/keyvault/) calls to use access policies (legacy).

##### Using Create Key Vault commands

When creating new key vaults by using API version 2026-02-01 or later, you must specify access policies configuration to avoid defaulting to Azure RBAC.

Make sure you have the latest version of the Azure CLI or PowerShell modules.

# [Azure CLI](#tab/azure-cli)

Update Azure CLI to the latest version. For more information, see [How to update the Azure CLI](/cli/azure/update-azure-cli).

# [PowerShell](#tab/azure-powershell)

Update your PowerShell modules to the latest version:
- [Update-Module (PowerShellGet)](/powershell/module/powershellget/update-module)
- [Update-AzModule (Az.Tools.Installer)](/powershell/module/az.tools.installer/update-azmodule)

---
Use the appropriate command to create a key vault with access policies:

# [Azure CLI](#tab/azure-cli)

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

##### Using Create Resource commands

When you create new key vaults by using API version 2026-02-01 or later, set `enableRbacAuthorization` to `false` to use access policies (legacy). If you don't specify this property, it defaults to `true` (Azure RBAC).

# [Azure CLI](#tab/azure-cli)

Use the [az resource create](/cli/azure/resource#az-resource-create) command and set `"enableRbacAuthorization": false` and `--api-version "2026-02-01"`:

```azurecli
az resource create --resource-group $resourceGroup --name $vaultName --resource-type "Microsoft.KeyVault/vaults" --location $location --api-version "2026-02-01" --properties "{\"sku\": { \"family\": \"A\", \"name\": \"standard\" }, \"tenantId\": \"$tenantID\",\"enableRbacAuthorization\": false, \"accessPolicies\": []}"
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
- [Use Azure RBAC for managing access to Key Vault](rbac-guide.md)
- [Secure your key vault](secure-key-vault.md)
- [Azure Key Vault REST API reference](/rest/api/keyvault/)

