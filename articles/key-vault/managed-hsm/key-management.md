---
title: Manage keys in a Managed HSM 
description: Use this article to manage keys in a managed HSM
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.custom: devx-track-azurecli
ms.topic: tutorial
ms.date: 04/13/2026

ms.author: mbaldwin
ai-usage: ai-assisted
---

# Manage keys in an Azure Managed HSM

> [!NOTE]
> Key Vault supports two types of resources: vaults and managed HSMs. This article is about **Managed HSM**. If you want to learn how to manage a vault, see [Quickstart: Create a key vault using the Azure CLI](../general/quick-create-cli.md).

For an overview of Managed HSM, see [What is Managed HSM?](overview.md)

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](~/reusable-content/ce-skilling/azure/includes/azure-subscription-prerequisite.md)]

You also need:

* A Managed HSM in your subscription. See [Quickstart: Provision and activate a managed HSM using the Azure portal](quick-create-portal.md) to provision and activate a Managed HSM.
* To use Azure CLI: Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).
* To use Azure PowerShell: The [Azure PowerShell module](/powershell/azure/install-azure-powershell).
* To access keys using Azure SDKs, see [.NET](quickstart-dotnet.md) | [Python](quickstart-python.md) | [JavaScript](quickstart-javascript.md)

> [!NOTE]
> All the following commands show two usage methods for CLI. One method uses the `--hsm-name` and `--name` (for key name) parameters. The other method uses the `--id` parameter, where you can specify the entire URL including the key name when appropriate. The latter method is useful when the caller (a user or an application) has no read access on the control plane and only restricted access on the data plane.

> [!NOTE]
> Some interactions with key material require specific Managed HSM local RBAC permissions. For a full list of built-in Managed HSM local RBAC roles and permissions, see [Managed HSM local RBAC built-in roles](./built-in-roles.md). To assign these permissions to a user, see [Secure access to your managed HSMs](./how-to-secure-access.md).
## Create an HSM key

> [!NOTE]
> You can't export a key that's generated or imported into Managed HSM. The only exception to the no-export rule is when you create a key with a specific [key release policy](../keys/policy-grammar.md). This policy allows the key to be exported only to trusted confidential computing environments (secure enclaves) that you explicitly define. This limited export capability is designed for specific secure computing scenarios and isn't the same as a general-purpose key export. For recommended best practices for key portability and durability, see the linked article.

# [Azure portal](#tab/azure-portal)

<!-- TODO: Fill in portal steps for key creation once portal experience is confirmed with Jack Richins. -->

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**.

1. Select **Generate/Import**.

1. Choose the key type (RSA-HSM, EC-HSM, or oct-HSM), set the key size or curve, name, and permitted operations, then select **Create**.

   [screenshot: Create key blade]

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key create` command to create a key.

### Create an RSA key

This example shows how to create a 3072-bit **RSA** key that is only used for **wrapKey** and **unwrapKey** operations (`--ops`).


```azurecli-interactive
az keyvault key create --hsm-name <hsm-name> --name myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072

## OR
# Note the key name (myrsakey) in the URI

az keyvault key create --id https://<hsm-name>.managedhsm.azure.net/keys/myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072
```

The `get` operation only returns the public key and key attributes. It doesn't return the private key (if an asymmetric key) or the key material (if a symmetric key).

### Create an EC key

The following example shows how to create an **EC** key with the P-256 curve. The key is only for **sign and verify** operations (--ops) and has two tags, **usage** and **appname**. Use tags to add extra metadata to the key for tracking and managing.

```azurecli-interactive
az keyvault key create --hsm-name <hsm-name> --name myec256key --ops sign verify  --tags 'usage=signing' 'appname=myapp' --kty EC-HSM --curve P-256

## OR
# Note the key name (myec256key) in the URI

az keyvault key create --id https://<hsm-name>.managedhsm.azure.net/keys/myec256key --ops sign verify  --tags 'usage=signing' 'appname=myapp' --kty EC-HSM --curve P-256
```

### Create a 256-bit symmetric key

This example shows how to create a 256-bit **symmetric** key that's only for **encrypt and decrypt** operations (--ops).

```azurecli-interactive
az keyvault key create --hsm-name <hsm-name> --name myaeskey --ops encrypt decrypt  --tags 'usage=encryption' 'appname=myapp' --kty oct-HSM --size 256

## OR
# Note the key name (myaeskey) in the URI

az keyvault key create --id https://<hsm-name>.managedhsm.azure.net/keys/myaeskey --ops encrypt decrypt  --tags 'usage=encryption' 'appname=myapp' --kty oct-HSM --size 256
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Add-AzKeyVaultKey` cmdlet to create a key.

### Create an RSA key

This example shows how to create a 3072-bit **RSA** key that is only used for **wrapKey** and **unwrapKey** operations.

```azurepowershell-interactive
Add-AzKeyVaultKey -HsmName <hsm-name> -Name myrsakey -KeyType RSA-HSM -Size 3072 -KeyOps wrapKey,unwrapKey
```

### Create an EC key

This example shows how to create an **EC** key with the P-256 curve for **sign and verify** operations.

```azurepowershell-interactive
Add-AzKeyVaultKey -HsmName <hsm-name> -Name myec256key -KeyType EC-HSM -CurveName P-256 -KeyOps sign,verify -Tag @{usage='signing'; appname='myapp'}
```

### Create a 256-bit symmetric key

This example shows how to create a 256-bit **symmetric** key for **encrypt and decrypt** operations.

```azurepowershell-interactive
Add-AzKeyVaultKey -HsmName <hsm-name> -Name myaeskey -KeyType oct-HSM -Size 256 -KeyOps encrypt,decrypt -Tag @{usage='encryption'; appname='myapp'}
```

---

## View key attributes and tags

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**.

1. Select the key you want to view. The portal displays the key's attributes, versions, and tags.

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key show` command to view attributes, versions, and tags for a key.

```azurecli-interactive
az keyvault key show --hsm-name <hsm-name> --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key show --id https://<hsm-name>.managedhsm.azure.net/keys/myrsakey

```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Get-AzKeyVaultKey` cmdlet to view attributes, versions, and tags for a key.

```azurepowershell-interactive
Get-AzKeyVaultKey -HsmName <hsm-name> -Name myrsakey
```

---

## List keys

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**. The portal lists all keys in the Managed HSM.

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key list` command to list all keys inside a managed HSM.

```azurecli-interactive
az keyvault key list --hsm-name <hsm-name>

## OR
# use full URI

az keyvault key list --id https://<hsm-name>.managedhsm.azure.net/

```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Get-AzKeyVaultKey` cmdlet to list all keys in a Managed HSM.

```azurepowershell-interactive
Get-AzKeyVaultKey -HsmName <hsm-name>
```

---

## Delete a key

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**.

1. Select the key you want to delete.

1. Select **Delete**, and then confirm.

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key delete` command to delete a key from a managed HSM. Soft-delete is always on. Therefore, a deleted key stays in the deleted state and you can recover it until the number of retention days pass. After that, the key is purged (permanently deleted) with no recovery possible.

```azurecli-interactive
az keyvault key delete --hsm-name <hsm-name> --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key delete --id https://<hsm-name>.managedhsm.azure.net/keys/myrsakey
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Remove-AzKeyVaultKey` cmdlet to delete a key. Soft-delete is always on, so a deleted key remains recoverable until the retention period expires.

```azurepowershell-interactive
Remove-AzKeyVaultKey -HsmName <hsm-name> -Name myrsakey
```

---

## List deleted keys

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**.

1. Select **Manage deleted keys** to view keys in the soft-deleted state.

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key list-deleted` command to list all the keys in the deleted state in your managed HSM.

```azurecli-interactive
az keyvault key list-deleted --hsm-name <hsm-name>

## OR
# use full URI

az keyvault key list-deleted --id https://<hsm-name>.managedhsm.azure.net/
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Get-AzKeyVaultKey` cmdlet with the `-InRemovedState` parameter to list deleted keys.

```azurepowershell-interactive
Get-AzKeyVaultKey -HsmName <hsm-name> -InRemovedState
```

---

## Recover (undelete) a deleted key

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**, then select **Manage deleted keys**.

1. Select the deleted key you want to recover.

1. Select **Recover**.

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key list-deleted` command to list all the keys in deleted state in your managed HSM. To recover (undelete) a key, use the `--id` parameter. You must note the `recoveryId` value of the deleted key obtained from the `az keyvault key list-deleted` command.

```azurecli-interactive
az keyvault key recover --hsm-name <hsm-name> --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://<hsm-name>.managedhsm.azure.net/deletedKeys/myrsakey
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Undo-AzKeyVaultKeyRemoval` cmdlet to recover a deleted key.

```azurepowershell-interactive
Undo-AzKeyVaultKeyRemoval -HsmName <hsm-name> -Name myrsakey
```

---

## Purge (permanently delete) a key

> [!NOTE]
> If the managed HSM has purge protection enabled, the purge operation isn't permitted. The key is automatically purged when the retention period passes.

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Keys**, then select **Manage deleted keys**.

1. Select the deleted key you want to purge.

1. Select **Purge**, and then confirm.

> [!WARNING]
> This operation permanently deletes your key.

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key purge` command to purge (permanently delete) a key.

```azurecli-interactive
az keyvault key purge --hsm-name <hsm-name> --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key purge --id https://<hsm-name>.managedhsm.azure.net/deletedKeys/myrsakey

```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Remove-AzKeyVaultKey` cmdlet with the `-InRemovedState` parameter to purge a deleted key.

```azurepowershell-interactive
Remove-AzKeyVaultKey -HsmName <hsm-name> -Name myrsakey -InRemovedState
```

> [!WARNING]
> This operation permanently deletes your key.

---

## Create a single key backup

> [!NOTE]
> Key backup, restore, and import operations are available only through the Azure CLI and Azure PowerShell.

# [Azure CLI](#tab/azure-cli)

Use `az keyvault key backup` to create a key backup. The backup file is an encrypted blob cryptographically tied to the Security Domain of the source HSM. You can only restore it in HSMs that share the same security domain. Read more about [Security Domain](security-domain.md).

```azurecli-interactive
az keyvault key backup --hsm-name <hsm-name> --name myrsakey --file myrsakey.backup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key backup --id https://<hsm-name>.managedhsm.azure.net/deletedKeys/myrsakey  --file myrsakey.backup

```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Backup-AzKeyVaultKey` cmdlet to create a key backup. The backup file is an encrypted blob cryptographically tied to the Security Domain of the source HSM.

```azurepowershell-interactive
Backup-AzKeyVaultKey -HsmName <hsm-name> -Name myrsakey -OutputFile myrsakey.backup
```

---

## Restore a single key from backup

# [Azure CLI](#tab/azure-cli)

Use `az keyvault key restore` to restore a single key. The source HSM where you created the backup must share the same security domain as the target HSM where you're restoring the key.

> [!NOTE]
> The restore operation fails if a key with the same name exists in active or deleted state.

```azurecli-interactive
az keyvault key restore --hsm-name <hsm-name> --name myrsakey --file myrsakey.backup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key restore --id https://<hsm-name>.managedhsm.azure.net/deletedKeys/myrsakey --file myrsakey.backup

```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Restore-AzKeyVaultKey` cmdlet to restore a single key. The source HSM where you created the backup must share the same security domain as the target HSM.

> [!NOTE]
> The restore operation fails if a key with the same name exists in active or deleted state.

```azurepowershell-interactive
Restore-AzKeyVaultKey -HsmName <hsm-name> -InputFile myrsakey.backup
```

---

## Import a key from a file

# [Azure CLI](#tab/azure-cli)

Use the `az keyvault key import` command to import a key (only RSA and EC) from a file. The certificate file must have a private key and must use PEM encoding (as defined in RFCs [1421](https://tools.ietf.org/html/rfc1421), [1422](https://tools.ietf.org/html/rfc1422), [1423](https://tools.ietf.org/html/rfc1423), [1424](https://tools.ietf.org/html/rfc1424)).

```azurecli-interactive
az keyvault key import --hsm-name <hsm-name> --name myrsakey --pem-file mycert.key --pem-password 'mypassword'

## OR
# Note the key name (<key-name>) in the URI

az keyvault key recover --id https://<hsm-name>.managedhsm.azure.net/deletedKeys/<key-name> --pem-file mycert.key --password 'mypassword'
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Add-AzKeyVaultKey` cmdlet with the `-KeyFilePath` parameter to import a key from a PEM file.

```azurepowershell-interactive
Add-AzKeyVaultKey -HsmName <hsm-name> -Name myrsakey -KeyFilePath ./mycert.key -KeyFilePassword (ConvertTo-SecureString -String 'mypassword' -AsPlainText -Force) -KeyType RSA-HSM
```

---

To import a key from your on-premises HSM to managed HSM, see [Import HSM-protected keys to Managed HSM (BYOK)](hsm-protected-keys-byok.md).

## Next steps

- For the complete Azure CLI reference for key vault commands, see [Key Vault CLI reference](/cli/azure/keyvault).
- For programming references, see [the Azure Key Vault developer's guide](../general/developers-guide.md).
- To access keys using Azure SDKs, see: [.NET](quickstart-dotnet.md), [Python](quickstart-python.md), [JavaScript](quickstart-javascript.md).
- To learn more about Managed HSM role management, see [Managed HSM role management](role-management.md).
- To learn more about securing your Azure Managed HSM deployment, see [Secure your Azure Managed HSM deployment](secure-managed-hsm.md).
