---
title: Manage keys in a managed HSM - Azure Key Vault | Microsoft Docs
description: Use this article to manage keys in a managed HSM
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.custom: devx-track-azurecli
ms.topic: tutorial
ms.date: 04/14/2025

ms.author: mbaldwin
---

# Manage a Managed HSM using the Azure CLI

> [!NOTE]
> Key Vault supports two types of resources: vaults and managed HSMs. This article is about **Managed HSM**. If you want to learn how to manage a vault, see [Manage Key Vault using the Azure CLI](../general/manage-with-cli2.md).

For an overview of Managed HSM, see [What is Managed HSM?](overview.md)

## Prerequisites

To complete the steps in this article, you must have the following items:

* A Microsoft Azure subscription. If you don't have one, you can sign up for a [free trial](https://azure.microsoft.com/pricing/free-trial).
* The Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI]( /cli/azure/install-azure-cli).
* A managed HSM in your subscription. See [Quickstart: Provision and activate a managed HSM using Azure CLI](quick-create-cli.md) to provision and activate a managed HSM.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Sign in to Azure

Sign in to Azure using the CLI by typing:

```azurecli
az login
```

For more information on sign in options via the CLI, see [sign in with Azure CLI](/cli/azure/authenticate-azure-cli)

> [!NOTE]
> All the commands following commands show two usage methods. One using `--hsm-name` and `--name` (for key name) parameters and another using `--id` parameter where you can specify the entire url including the key name where appropriate. The latter method is useful when the caller (a user or an application) has no read access on the control plane and only restricted access on the data plane.

> [!NOTE]
> Some interactions with key material require specific Local RBAC permissions. For a full list of built-in Local RBAC roles and permissions, see [Managed HSM local RBAC built-in roles](./built-in-roles.md). To assign these permissions to a user, see [Secure access to your managed HSMs](./how-to-secure-access.md)
## Create an HSM key

> [!NOTE]
> A key that's generated or imported into Managed HSM cannot be exported. The only exception to the no-export rule is when you create a key with a specific [key release policy](../keys/policy-grammar.md). This policy allows the key to be exported only to trusted confidential computing environments (secure enclaves) that you explicitly define. This limited export capability is designed for specific secure computing scenarios and is not the same as a general-purpose key export. Refer to recommended best practices for key portability and durability.

Use `az keyvault key create` command to create a key.

### Create an RSA key

This example shows how to create a 3072-bit **RSA** key that is only used for **wrapKey, unwrapKey** operations (--ops).


```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072

## OR
# Note the key name (myrsakey) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey --ops wrapKey unwrapKey --kty RSA-HSM --size 3072
```

Note, that the `get` operation only returns the public key and key attributes. It does not return the private key (if a asymmetric key) or the key material (if a symmetric key).

### Create an EC key

The example below shows how to create an **EC** key with P-256 curve that will be only used for **sign and verify** operations (--ops) and has two tags, **usage** and **appname**. Tags help you add additional metadata to the key for tracking and managing.

```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myec256key --ops sign verify  --tags ‘usage=signing] appname=myapp’ --kty EC-HSM --curve P-256

## OR
# Note the key name (myec256key) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myec256key --ops sign verify  --tags ‘usage=signing] appname=myapp’ --kty EC-HSM --curve P-256
```

### Create a 256-bit symmetric key

This example shows how to create a 256-bit **symmetric** key that will be only used for **encrypt and decrypt** operations (--ops).

```azurecli-interactive
az keyvault key create --hsm-name ContosoMHSM --name myaeskey --ops encrypt decrypt  --tags --kty oct-HSM --size 256

## OR
# Note the key name (myaeskey) in the URI

az keyvault key create --id https://ContosoMHSM.managedhsm.azure.net/keys/myaeskey --ops encrypt decrypt  --tags ‘usage=signing] appname=myapp’ --kty oct-HSM --size 256
```

## View key attributes and tags

Use `az keyvault key show` command to view attributes, versions, and tags for a key.

```azurecli-interactive
az keyvault key show --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key show --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey

```

## List keys

Use `az keyvault key list` command to list all keys inside a managed HSM.

```azurecli-interactive
az keyvault key list --hsm-name ContosoHSM

## OR
# use full URI

az keyvault key list --id https://ContosoMHSM.managedhsm.azure.net/

```

## Delete a key

Use `az keyvault key delete` command to delete a key from a managed HSM. Soft-delete is always on. Therefore a deleted key remains in deleted state and can be recovered until the number of retention days have passed, at which point the key is purged (permanently deleted) with no recovery possible.

```azurecli-interactive
az keyvault key delete --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key delete --id https://ContosoMHSM.managedhsm.azure.net/keys/myrsakey
```

## List deleted keys

Use `az keyvault key list-deleted` command to list all the keys in deleted state in your managed HSM.

```azurecli-interactive
az keyvault key list-deleted --hsm-name ContosoHSM

## OR
# use full URI

az keyvault key list-deleted --id https://ContosoMHSM.managedhsm.azure.net/
```

## Recover (undelete) a deleted key

Use `az keyvault key list-deleted` command to list all the keys in deleted state in your managed HSM. If you need to recover (undelete) a key using the `--id` parameter while recovering a deleted key, you must note the `recoveryId` value of the deleted key obtained from the `az keyvault key list-deleted` command.

```azurecli-interactive
az keyvault key recover --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey
```

## Purge (permanently delete) a key

Use `az keyvault key purge` command to purge (permanently delete) a key.

> [!NOTE]
> If the managed HSM has purge protection enabled, purge operation is not permitted. The key is automatically purged when the retention period has passed.

```azurecli-interactive
az keyvault key purge --hsm-name ContosoHSM --name myrsakey

## OR
# Note the key name (myaeskey) in the URI

az keyvault key purge --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey

```

## Create a single key backup

Use `az keyvault key backup` to create a key backup. The backup file is an encrypted blob cryptographically tied to the Security Domain of the source HSM. It can only be restored in HSMs that share the same security domain. Read more about [Security Domain](security-domain.md).

```azurecli-interactive
az keyvault key backup --hsm-name ContosoHSM --name myrsakey --file myrsakey.backup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key backup --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey  --file myrsakey.backup

```

## Restore a single key from backup

Use `az keyvault key restore` to restore a single key. The source HSM where the backup was created must share the same security domain as the target HSM where the key is being restored.

> [!NOTE]
> The restore will not succeed if a key with same name exists in active or deleted state.

```azurecli-interactive
az keyvault key restore --hsm-name ContosoHSM --name myrsakey --file myrsakey.backup

## OR
# Note the key name (myaeskey) in the URI

az keyvault key restore --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey --file myrsakey.backup

```

## Import a key from a file

Use `az keyvault key import` command to import a key (only RSA and EC) from a file. The certificate file must have private key and must use PEM encoding (as defined in RFCs [1421](https://tools.ietf.org/html/rfc1421), [1422](https://tools.ietf.org/html/rfc1422), [1423](https://tools.ietf.org/html/rfc1423), [1424](https://tools.ietf.org/html/rfc1424)).

```azurecli-interactive
az keyvault key import --hsm-name ContosoHSM --name myrsakey --pem-file mycert.key --pem-password 'mypassword'

## OR
# Note the key name (myaeskey) in the URI

az keyvault key recover --id https://ContosoMHSM.managedhsm.azure.net/deletedKeys/myrsakey --pem-file mycert.key --password 'mypassword'
```

To import a key from your on-premises HSM to managed HSM, see [Import HSM-protected keys to Managed HSM (BYOK)](hsm-protected-keys-byok.md)

## Next steps

- For complete Azure CLI reference for key vault commands, see [Key Vault CLI reference](/cli/azure/keyvault).
- For programming references, see [the Azure Key Vault developer's guide](../general/developers-guide.md)
- Learn more about [Managed HSM role management](role-management.md)
- Learn more about [Managed HSM best practices](best-practices.md)
