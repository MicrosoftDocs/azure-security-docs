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

> [!IMPORTANT]
> Starting February 2026, Azure Key Vault API version 2026-02-01 establishes Azure RBAC as the default access control model. This is a **breaking change** that affects all new key vault creation unless you explicitly configure access policies.

Azure Key Vault is implementing an important security enhancement in API version 2026-02-01, releasing in February 2026. To help protect your key vaults and reduce security risks, **the new Key Vault API version establishes Azure RBAC as the default access control model**, consistent with the Azure portal experience.

The ramifications of this change are as follows:

- **New key vaults created with API version 2026-02-01**: All new key vaults created with API version 2026-02-01 have the `enableRbacAuthorization` property explicitly set. If you don't specify a value when creating a vault, it defaults to Azure RBAC (`enableRbacAuthorization` = `true`). To use access policies, you must explicitly set `enableRbacAuthorization` to `false` during creation.
 
- **Existing key vaults with `enableRbacAuthorization` set to `true` or `false`**: Existing key vaults that are explicitly set to either Azure RBAC (`true`) or access policies (`false`) continue using their current access control model. 
 
- **Existing key vaults with undefined `enableRbacAuthorization`**: Key vaults with an undefined `enableRbacAuthorization` property were created with an API version earlier than 2026-02-01 and are treated as using access policies (`enableRbacAuthorization` = `false`). These vaults can be migrated to Azure RBAC by explicitly setting `enableRbacAuthorization` to `true`.

> [!IMPORTANT]
> To change the `enableRbacAuthorization` property for a key vault, you must have the `Microsoft.Authorization/roleAssignments/write` permission. This permission is included in roles such as Owner and User Access Administrator. For more information, see [Enable Azure RBAC permissions on Key Vault](rbac-guide.md#enable-azure-rbac-permissions-on-key-vault).
 
We strongly recommend that key vaults currently using access policies migrate to Azure RBAC for improved security. For more information on why we recommend Azure RBAC, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md).

Follow the steps in the following section to check your current configuration and either migrate to Azure RBAC (recommended) or continue using access policies (legacy).

> [!WARNING]
> All API versions before 2026-02-01 retire on February 27, 2027. Complete these steps by that date to avoid service disruption.

## Step 1: Check current configurations

Check if your vault's access configuration is set to Azure RBAC or access policies.

After checking your configuration:
- If your vaults use **Azure RBAC** (`enableRbacAuthorization` = `true`), go to [Step 2: For vaults already using Azure RBAC](#step-2-for-vaults-already-using-azure-rbac)
- If your vaults use **access policies** (`enableRbacAuthorization` = `false` or undefined), go to [Step 3: For vaults using access policies](#step-3-for-vaults-using-access-policies)

> [!NOTE]
> Key vaults with an undefined `enableRbacAuthorization` property were created with an API version earlier than 2026-02-01 and are treated as using access policies.

### Check a single vault

# [Azure CLI](#tab/azure-cli)

1. Use the [az keyvault show](/cli/azure/keyvault#az-keyvault-show) command to retrieve vault details:

   ```azurecli
   az keyvault show --name <KeyVaultName> --resource-group <ResourceGroupName>
   ```

1. Check the **Enabled for RBAC Authorization** property for the key vault.

# [PowerShell](#tab/azure-powershell)

1. Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to get vault details:

   ```azurepowershell
   Get-AzKeyVault -ResourceGroupName $resourceGroupName -VaultName $keyVaultName
   ```

1. Check the **Enabled for RBAC Authorization** property for the key vault.

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

If your key vaults use access policies (`enableRbacAuthorization` = `false` or undefined), decide if you want to migrate to role-based access (recommended) or continue using access policies. For more information on access control models, see [Use Azure RBAC for managing access to Key Vault](rbac-guide.md) and [Azure Key Vault best practices](secure-key-vault.md).

> [!NOTE]


**Choose your path:**
- **Recommended**: Go to [Step 4: Migrate to Azure RBAC](#step-4-migrate-to-azure-rbac-recommended)
- **Legacy**: Go to [Step 5: Continue using access policies](#step-5-continue-using-access-policies)

## Step 4: Migrate to Azure RBAC (recommended)

Use this opportunity to increase your security posture by migrating from vault access policy to Azure RBAC for managing access. For detailed migration guidance, see [Migrate from vault access policy to an Azure role-based access control permission model](rbac-migration.md).

Update all Key Vault ARM, BICEP, Terraform templates, and REST API calls to use API version 2026-02-01.

## Step 5: Continue using access policies

To continue using access policies, follow the instructions in this section. 



Choose one of the following methods based on your scenario:
- [For existing vaults with undefined access configuration](#for-existing-vaults-with-undefined-access-configuration) - Optionally update existing vaults to explicitly use access policies
- [Using ARM, BICEP, Terraform templates](#using-arm-bicep-terraform-templates) - Create new vaults or update existing vaults to explicitly use access policies
- [Using Create Key Vault commands](#using-create-key-vault-commands) - Create new vaults to explicitly use access policies
- [Using Create Resource commands](#using-create-resource-commands) - Create new vaults to explicitly use access policies

#### For existing vaults with undefined access configuration

Key vaults with an undefined `enableRbacAuthorization` property were created with an API version earlier than 2026-02-01 and are already treated as using access policies. These vaults will continue to function with access policies without any changes required.

If you want to explicitly set the `enableRbacAuthorization` property to `false` for clarity, you can do so using the following commands. This is optional and does not change the vault's behavior.

> [!IMPORTANT]
> To change the `enableRbacAuthorization` property for a key vault, you must have the `Microsoft.Authorization/roleAssignments/write` permission. This permission is included in roles such as Owner and User Access Administrator. For more information, see [Enable Azure RBAC permissions on Key Vault](rbac-guide.md#enable-azure-rbac-permissions-on-key-vault).

# [Azure CLI](#tab/azure-cli)

Use the [az keyvault update](/cli/azure/keyvault#az-keyvault-update) command and set `--enable-rbac-authorization false`:

```azurecli
az keyvault update --name <KeyVaultName> --resource-group <ResourceGroupName> --enable-rbac-authorization false
```

# [PowerShell](#tab/azure-powershell)

Use the [Update-AzKeyVault](/powershell/module/az.keyvault/update-azkeyvault) cmdlet and set `-EnableRbacAuthorization $false`:

```azurepowershell
Update-AzKeyVault -ResourceGroupName <ResourceGroupName> -VaultName <KeyVaultName> -EnableRbacAuthorization $false
```

---

#### Using ARM, BICEP, Terraform templates

When creating new key vaults with API version 2026-02-01 or updating existing vaults, explicitly set `enableRbacAuthorization` to `false` in all Key Vault ARM, BICEP, Terraform templates, and [REST API](/rest/api/keyvault/) calls to use access policies.


#### Using Create Key Vault commands

When creating new key vaults with API version 2026-02-01, you must explicitly specify access policies configuration to avoid defaulting to Azure RBAC.

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

#### Using Create Resource commands

When creating new key vaults with API version 2026-02-01, you must explicitly set `enableRbacAuthorization` to `false` to use access policies. If you don't specify this property, it defaults to `true` (Azure RBAC).

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
- [Secure your key vault](secure-key-vault.md)
- [Azure Key Vault REST API reference](/rest/api/keyvault/)

