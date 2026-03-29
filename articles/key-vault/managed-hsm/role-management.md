---
title: Managed HSM data plane role management
description: Use this article to manage role assignments for your managed HSM. 
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.custom: devx-track-azurecli
ms.topic: tutorial
ms.date: 04/15/2025
ms.author: mbaldwin
---
# Managed HSM role management

> [!NOTE]
> Key Vault supports two types of resources: vaults and managed HSMs. This article is about **Managed HSM**. To learn how to manage a vault, see [Quickstart: Create a key vault using the Azure CLI](../general/quick-create-cli.md).

This article provides practical instructions for managing roles and role assignments for a Managed HSM by using the Azure CLI. It implements the role-based access control model described in [Access control for Managed HSM](access-control.md) by using the built-in roles documented in [Local RBAC built-in roles for Managed HSM](built-in-roles.md).

For an overview of Managed HSM, see [What is Managed HSM?](overview.md) If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

To allow a security principal (such as a user, a service principal, group, or a managed identity) to perform managed HSM data plane operations, assign them a role that permits those operations. For example, if you want to allow an application to perform a sign operation by using a key, assign it a role that contains the `Microsoft.KeyVault/managedHSM/keys/sign/action` as one of the data actions. Assign a role at a specific scope. Managed HSM local RBAC supports two scopes, HSM-wide (`/` or `/keys`) and per key (`/keys/<key-name>`).

For a list of all Managed HSM built-in roles and the operations they permit, see [Managed HSM built-in roles](built-in-roles.md).

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](../includes/azure-subscription-prerequisite.md)]

You also need:

* Azure CLI version 2.25.0 or later. Run `az --version` to find the version. To install or upgrade, see [Install the Azure CLI]( /cli/azure/install-azure-cli).
* A managed HSM in your subscription. To provision and activate a managed HSM, see [Quickstart: Provision and activate a managed HSM using Azure CLI](quick-create-cli.md).

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Sign in to Azure

To sign in to Azure using the CLI you can type:

```azurecli
az login
```

For more information on sign in options via the CLI, see [sign in with Azure CLI](/cli/azure/authenticate-azure-cli)

## Create a new role assignment

### Assign roles for all keys

Use `az keyvault role assignment create` command to assign a **Managed HSM Crypto User** role to user identified by user principal name `<user-principal-name>` for all  **keys** (scope `/keys`) in the `<hsm-name>`.

```azurecli-interactive
az keyvault role assignment create --hsm-name <hsm-name> --role "Managed HSM Crypto User" --assignee <user-principal-name>  --scope /keys
```

### Assign role for a specific key

Use `az keyvault role assignment create` command to assign a **Managed HSM Crypto User** role to user identified by user principal name `<user-principal-name>` for a specific key named `<key-name>`.

```azurecli-interactive
az keyvault role assignment create --hsm-name <hsm-name> --role "Managed HSM Crypto User" --assignee <user-principal-name>  --scope /keys/<key-name>
```

## List existing role assignments

Use `az keyvault role assignment list` to list role assignments.

All role assignments at scope / (default when no --scope is specified) for all users (default when no --assignee is specified)

```azurecli-interactive
az keyvault role assignment list --hsm-name <hsm-name>
```

All the role assignments at the HSM level for a specific user `<user-principal-name>`.

```azurecli-interactive
az keyvault role assignment list --hsm-name <hsm-name> --assignee <user-principal-name>
```

> [!NOTE]
> When scope is / (or /keys) the list command only lists all the role assignments at the top level and does not show role assignments at individual key level.

All role assignments for a specific user `<user-principal-name>` for a specific key `<key-name>`.

```azurecli-interactive
az keyvault role assignment list --hsm-name <hsm-name> --assignee <user-principal-name> --scope /keys/<key-name>
```

A specific role assignment for role **Managed HSM Crypto Officer** for a specific user `<user-principal-name>` for a specific key `<key-name>`


```azurecli-interactive
az keyvault role assignment list --hsm-name <hsm-name> --assignee <user-principal-name> --scope /keys/<key-name> --role "Managed HSM Crypto Officer"
```

## Delete a role assignment

Use `az keyvault role assignment delete` command to delete a **Managed HSM Crypto Officer** role assigned to user `<user-principal-name>` for key `<key-name>`.

```azurecli-interactive
az keyvault role assignment delete --hsm-name <hsm-name> --role "Managed HSM Crypto Officer" --assignee <user-principal-name>  --scope /keys/<key-name>
```

## List all available role definitions

Use `az keyvault role definition list` command to list all the role definitions.

```azurecli-interactive
az keyvault role definition list --hsm-name <hsm-name>
```

## Create a new role definition

Managed HSM has several built-in (pre-defined) roles that are useful for most common usage scenarios. You can define your own role with a list of specific actions that the role is allowed to perform. Then you can assign this role to principals to grant them the permission to the specified actions. 

Use `az keyvault role definition create` command to a role named **My Custom Role** using a JSON string.
```azurecli-interactive
az keyvault role definition create --hsm-name <hsm-name> --role-definition '{
    "roleName": "My Custom Role",
    "description": "The description of the custom rule.",
    "actions": [],
    "notActions": [],
    "dataActions": [
        "Microsoft.KeyVault/managedHsm/keys/read/action"
    ],
    "notDataActions": []
}'
```

Use `az keyvault role definition create` command to a role from a file named **my-custom-role-definition.json** containing the JSON string for a role definition. See example above.
```azurecli-interactive
az keyvault role definition create --hsm-name <hsm-name> --role-definition @my-custom-role-definition.json
```

## Show details of a role definition

Use `az keyvault role definition show` command to see details of a specific role definition using name (a GUID).

```azurecli-interactive
az keyvault role definition show --hsm-name <hsm-name> --name xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

## Update a custom role definition

Use `az keyvault role definition update` command to update a role named **My Custom Role** using a JSON string.
```azurecli-interactive
az keyvault role definition create --hsm-name <hsm-name> --role-definition '{
            "roleName": "My Custom Role",
            "name": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "id": "Microsoft.KeyVault/providers/Microsoft.Authorization/roleDefinitions/xxxxxxxx-
        xxxx-xxxx-xxxx-xxxxxxxxxxxx",
            "description": "The description of the custom rule.",
            "actions": [],
            "notActions": [],
            "dataActions": [
                "Microsoft.KeyVault/managedHsm/keys/read/action",
                "Microsoft.KeyVault/managedHsm/keys/write/action",
                "Microsoft.KeyVault/managedHsm/keys/backup/action",
                "Microsoft.KeyVault/managedHsm/keys/create"
            ],
            "notDataActions": []
        }'
```

## Delete custom role definition

Use the Azure CLI [az keyvault role definition delete](/cli/azure/keyvault/role/definition#az-keyvault-role-definition-delete) command to delete a custom role definition using name (a GUID).

```azurecli-interactive
az keyvault role definition delete --hsm-name <hsm-name> --name xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

> [!NOTE]
> Built-in roles cannot be deleted. When custom roles are deleted, all the role assignments using that custom role become defunct.

## Next steps

- See an overview of [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview).
- Learn more about [Managed HSM access control model](access-control.md)
- See all the [built-in roles for Managed HSM local RBAC](built-in-roles.md)
