---
title: Azure Key Vault recovery overview | Microsoft Docs
description: Key Vault Recovery features are designed to prevent the accidental or malicious deletion of your key vault and secrets, keys, and certificate stored inside key-vault.
ms.service: azure-key-vault
ms.subservice: general
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.topic: how-to
ms.author: mbaldwin
author: msmbaldwin
ms.date: 02/20/2024
---

# Azure Key Vault recovery management with soft delete and purge protection

This article covers two recovery features of Azure Key Vault, soft delete and purge protection. This document provides an overview of these features, and shows you how to manage them through the Azure portal, Azure CLI, and Azure PowerShell.

> [!IMPORTANT]
> If a key vault does not have soft-delete protection enabled, deleting a key deletes it permanently. Customers are strongly encouraged to turn on soft delete enforcement for their vaults via [Azure Policy](../policy-reference.md).

For more information about Key Vault, see
- [Key Vault overview](overview.md)
- [Azure Key Vault keys, secrets, and certificates overview](about-keys-secrets-certificates.md)

## Prerequisites

* An Azure subscription - [create one for free](https://azure.microsoft.com/free/dotnet)
* [Azure PowerShell](/powershell/azure/install-azure-powershell).
* [Azure CLI](/cli/azure/install-azure-cli)
* A Key Vault - you can create one using [Azure portal](../general/quick-create-portal.md) [Azure CLI](../general/quick-create-cli.md), or [Azure PowerShell](../general/quick-create-powershell.md)
* The user needs the following permissions (at subscription level) to perform operations on soft-deleted vaults:

  | Permission | Description |
  |---|---|
  |Microsoft.KeyVault/locations/deletedVaults/read|View the properties of a soft deleted key vault|
  |Microsoft.KeyVault/locations/deletedVaults/purge/action|Purge a soft deleted key vault|
  |Microsoft.KeyVault/locations/operationResults/read| To check purging state of vault|
  |[Key Vault Contributor](/azure/role-based-access-control/built-in-roles#key-vault-contributor)|To recover soft-deleted vault|

## What are soft-delete and purge protection

[Soft delete](soft-delete-overview.md) and purge protection are two different key vault recovery features.

**Soft delete** is designed to prevent accidental deletion of your key vault and keys, secrets, and certificates stored inside key vault. Think of soft-delete like a recycle bin. When you delete a key vault or a key vault object, it remains recoverable for a user configurable retention period or a default of 90 days. Key vaults in the soft deleted state can also be **purged** (permanently deleted), allowing you to recreate key vaults and key vault objects with the same name. Both recovering and deleting key vaults and objects require elevated access policy permissions. **Once soft delete has been enabled, it cannot be disabled.**

It is important to note that **key vault names are globally unique**, so you aren't able to create a key vault with the same name as a key vault in the soft deleted state. Similarly, the names of keys, secrets, and certificates are unique within a key vault. You aren't be able to create a secret, key, or certificate with the same name as another in the soft deleted state.

**Purge protection** is designed to prevent the deletion of your key vault, keys, secrets, and certificates by a malicious insider. Think of it as a recycle bin with a time based lock. You can recover items at any point during the configurable retention period. **You will not be able to permanently delete or purge a key vault until the retention period elapses.** Once the retention period elapses the key vault or key vault object is purged automatically.

> [!NOTE]
> Purge Protection is designed so that no administrator role or permission can override, disable, or circumvent purge protection. **When purge protection is enabled, it cannot be disabled or overridden by anyone including Microsoft.** This means you must recover a deleted key vault or wait for the retention period to elapse before reusing the key vault name.

For more information about soft-delete, see [Azure Key Vault soft-delete overview](soft-delete-overview.md)

# [Azure portal](#tab/azure-portal)

## Verify if soft delete is enabled on a key vault and enable soft delete

1. Sign in to the Azure portal.
1. Select your key vault.
1. Select the "Properties" blade.
1. Verify if the radio button next to soft-delete is set to "Enable Recovery".
1. If soft-delete is not enabled on the key vault, select the radio button to enable soft delete and select "Save".

:::image type="content" source="../media/key-vault-recovery-1.png" alt-text="On Properties, Soft-delete is highlighted, as is the value to enable it.":::

## Grant access to a service principal to purge and recover deleted secrets

1. Sign in to the Azure portal.
1. Select your key vault.
1. Select the "Access Policy" blade.
1. In the table, find the row of the security principal you wish to grant access to (or add a new security principal).
1. Select the drop-down for keys, certificates, and secrets.
1. Scroll to the bottom of the drop-down and select "Recover" and "Purge"
2. Security principals also need "get" and "list" functionality to perform most operations.

:::image type="content" source="../media/key-vault-recovery-2.png" alt-text="In the left navigation pane, Access policies is highlighted. On Access policies, the Secret Positions drop-down list is shown, and four items are selected: Get, List, Recover, and Purge.":::

## List, recover, or purge a soft-deleted key vault

1. Sign in to the Azure portal.
1. Select the search bar at the top of the page.
1. Search for the "Key Vault" service. Do not select an individual key vault.
1. At the top of the screen, select the option to "Manage deleted vaults"
1. A context pane opens on the right side of your screen.
1. Select your subscription.
1. If your key vault has been soft deleted, it appears in the context pane on the right.
1. If there are too many vaults, you can either select "Load More" at the bottom of the context pane or use CLI or PowerShell to get the results.
1. Once you find the vault you wish to recover or purge, select the checkbox next to it.
1. Select the recover option at the bottom of the context pane if you would like to recover the key vault.
1. Select the purge option if you would like to permanently delete the key vault.

:::image type="content" source="../media/key-vault-recovery-3.png" alt-text="On Key vaults, the Manage deleted vaults option is highlighted.":::

:::image type="content" source="../media/key-vault-recovery-4.png" alt-text="On Manage deleted key vaults, the only listed key vault is highlighted and selected, and the Recover button is highlighted.":::

## List, recover or purge soft deleted secrets, keys, and certificates

1. Sign in to the Azure portal.
1. Select your key vault.
1. Select the blade corresponding to the secret type you want to manage (keys, secrets, or certificates).
1. At the top of the screen, select "Manage deleted (keys, secrets, or certificates)
1. A context pane appears on the right side of your screen.
1. If your secret, key, or certificate does not appear in the list, it is not in the soft-deleted state.
1. Select the secret, key, or certificate you would like to manage.
1. Select the option to recover or purge at the bottom of the context pane.

:::image type="content" source="../media/key-vault-recovery-5.png" alt-text="On Keys, the Manage deleted keys option is highlighted.":::

# [Azure CLI](#tab/azure-cli)

## Key Vault (CLI)

* Verify if a key-vault has soft-delete enabled

    ```azurecli
    az keyvault show --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}
    ```

* Enable soft-delete on key-vault

    All new key vaults have soft delete enabled by default. If you currently have a key vault that does not have soft delete enabled, use the following command to enable soft delete.

    ```azurecli
    az keyvault update --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME} --enable-soft-delete true
    ```

* Delete key vault (recoverable if soft delete is enabled)

    ```azurecli
    az keyvault delete --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}
    ```

* List all soft-deleted key vaults

    ```azurecli
    az keyvault list-deleted --subscription {SUBSCRIPTION ID} --resource-type vault
    ```

* Recover soft-deleted key-vault

    ```azurecli
    az keyvault recover --subscription {SUBSCRIPTION ID} -n {VAULT NAME}
    ```

* Purge soft-deleted key vault **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR KEY VAULT)**

    ```azurecli
    az keyvault purge --subscription {SUBSCRIPTION ID} -n {VAULT NAME}
    ```

* Enable purge-protection on key-vault

    ```azurecli
    az keyvault update --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME} --enable-purge-protection true
    ```

## Certificates (CLI)

* Grant access to purge and recover certificates

    ```azurecli
    az keyvault set-policy --upn user@contoso.com --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}  --certificate-permissions recover purge
    ```

* Delete certificate

    ```azurecli
    az keyvault certificate delete --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {CERTIFICATE NAME}
    ```

* List deleted certificates

    ```azurecli
    az keyvault certificate list-deleted --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME}
    ```

* Recover deleted certificate

    ```azurecli
    az keyvault certificate recover --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {CERTIFICATE NAME}
    ```

* Purge soft-deleted certificate **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR CERTIFICATE)**

    ```azurecli
    az keyvault certificate purge --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {CERTIFICATE NAME}
    ```

## Keys (CLI)

* Grant access to purge and recover keys

    ```azurecli
    az keyvault set-policy --upn user@contoso.com --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}  --key-permissions recover purge
    ```

* Delete key

    ```azurecli
    az keyvault key delete --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {KEY NAME}
    ```

* List deleted keys

    ```azurecli
    az keyvault key list-deleted --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME}
    ```

* Recover deleted key

    ```azurecli
    az keyvault key recover --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {KEY NAME}
    ```

* Purge soft-deleted key **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR KEY)**

    ```azurecli
    az keyvault key purge --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {KEY NAME}
    ```

## Secrets (CLI)

* Grant access to purge and recover secrets

    ```azurecli
    az keyvault set-policy --upn user@contoso.com --subscription {SUBSCRIPTION ID} -g {RESOURCE GROUP} -n {VAULT NAME}  --secret-permissions recover purge
    ```

* Delete secret

    ```azurecli
    az keyvault secret delete --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {SECRET NAME}
    ```

* List deleted secrets

    ```azurecli
    az keyvault secret list-deleted --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME}
    ```

* Recover deleted secret

    ```azurecli
    az keyvault secret recover --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {SECRET NAME}
    ```

* Purge soft-deleted secret **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR SECRET)**

    ```azurecli
    az keyvault secret purge --subscription {SUBSCRIPTION ID} --vault-name {VAULT NAME} --name {SECRET NAME}
    ```

# [Azure PowerShell](#tab/azure-powershell)

## Key Vault (PowerShell)

* Verify if a key-vault has soft-delete enabled

    ```azurepowershell
    Get-AzKeyVault -VaultName "ContosoVault"
    ```

* Delete key vault

    ```azurepowershell
    Remove-AzKeyVault -VaultName 'ContosoVault'
    ```

* List all soft-deleted key vaults

    ```azurepowershell
    Get-AzKeyVault -InRemovedState
    ```

* Recover soft-deleted key-vault

    ```azurepowershell
    Undo-AzKeyVaultRemoval -VaultName ContosoVault -ResourceGroupName ContosoRG -Location westus
    ```

* Purge soft-deleted key-vault **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR KEY VAULT)**

    ```azurepowershell
    Remove-AzKeyVault -VaultName ContosoVault -InRemovedState -Location westus
    ```

* Enable purge-protection on key-vault

    ```azurepowershell
    Update-AzKeyVault -VaultName ContosoVault -ResourceGroupName ContosoRG -EnablePurgeProtection
    ```

## Certificates (PowerShell)

* Grant permissions to recover and purge certificates

    ```azurepowershell
    Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToCertificates recover,purge
    ```

* Delete a Certificate

  ```azurepowershell
  Remove-AzKeyVaultCertificate -VaultName ContosoVault -Name 'MyCert'
  ```

* List all deleted certificates in a key vault

  ```azurepowershell
  Get-AzKeyVaultCertificate -VaultName ContosoVault -InRemovedState
  ```

* Recover a certificate in the deleted state

  ```azurepowershell
  Undo-AzKeyVaultCertificateRemoval -VaultName ContosoVault -Name 'MyCert'
  ```

* Purge a soft-deleted certificate **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR CERTIFICATE)**

  ```azurepowershell
  Remove-AzKeyVaultcertificate -VaultName ContosoVault -Name 'MyCert' -InRemovedState
  ```

## Keys (PowerShell)

* Grant permissions to recover and purge keys

    ```azurepowershell
    Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToKeys recover,purge
    ```

* Delete a key

  ```azurepowershell
  Remove-AzKeyVaultKey -VaultName ContosoVault -Name 'MyKey'
  ```

* List all deleted keys in a key vault

  ```azurepowershell
  Get-AzKeyVaultKey -VaultName ContosoVault -InRemovedState
  ```

* To recover a soft-deleted key

    ```azurepowershell
    Undo-AzKeyVaultKeyRemoval -VaultName ContosoVault -Name ContosoFirstKey
    ```

* Purge a soft-deleted key **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR KEY)**

    ```azurepowershell
    Remove-AzKeyVaultKey -VaultName ContosoVault -Name ContosoFirstKey -InRemovedState
    ```

## Secrets (PowerShell)

* Grant permissions to recover and purge secrets

    ```azurepowershell
    Set-AzKeyVaultAccessPolicy -VaultName ContosoVault -UserPrincipalName user@contoso.com -PermissionsToSecrets recover,purge
    ```

* Delete a secret named SQLPassword

  ```azurepowershell
  Remove-AzKeyVaultSecret -VaultName ContosoVault -Name SQLPassword
  ```

* List all deleted secrets in a key vault

  ```azurepowershell
  Get-AzKeyVaultSecret -VaultName ContosoVault -InRemovedState
  ```

* Recover a secret in the deleted state

  ```azurepowershell
  Undo-AzKeyVaultSecretRemoval -VaultName ContosoVault -Name SQLPassword
  ```

* Purge a secret in deleted state **(WARNING! THIS OPERATION WILL PERMANENTLY DELETE YOUR KEY)**

  ```azurepowershell
  Remove-AzKeyVaultSecret -VaultName ContosoVault -Name SQLPassword -InRemovedState 
  ```
---

## Next steps

- [Azure Key Vault PowerShell cmdlets](/powershell/module/az.keyvault)
- [Key Vault Azure CLI commands](/cli/azure/keyvault)
- [Azure Key Vault backup](backup.md)
- [How to enable Key Vault logging](howto-logging.md)
- [Azure Key Vault security features](security-features.md)
- [Azure Key Vault developer's guide](developers-guide.md)
