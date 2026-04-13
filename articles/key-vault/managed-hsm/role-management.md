---
title: Managed HSM data plane role management
description: Use this article to manage role assignments for your managed HSM. 
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
# Managed HSM role management

> [!NOTE]
> Key Vault supports two types of resources: vaults and managed HSMs. This article is about **Managed HSM**. To learn how to manage a vault, see [Quickstart: Create a key vault using the Azure CLI](../general/quick-create-cli.md).

This article provides practical instructions for managing roles and role assignments for a Managed HSM. It implements the role-based access control model described in [Access control for Managed HSM](access-control.md) by using the built-in roles documented in [Local RBAC built-in roles for Managed HSM](built-in-roles.md).

For an overview of Managed HSM, see [What is Managed HSM?](overview.md) If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

To allow a security principal (such as a user, a service principal, group, or a managed identity) to perform managed HSM data plane operations, assign them a role that permits those operations. For example, if you want to allow an application to perform a sign operation by using a key, assign it a role that contains the `Microsoft.KeyVault/managedHSM/keys/sign/action` as one of the data actions. Assign a role at a specific scope. Managed HSM local RBAC supports two scopes, HSM-wide (`/` or `/keys`) and per key (`/keys/<key-name>`).

For a list of all Managed HSM built-in roles and the operations they permit, see [Managed HSM built-in roles](built-in-roles.md).

## Prerequisites

[!INCLUDE [Azure subscription prerequisite](~/reusable-content/ce-skilling/azure/includes/azure-subscription-prerequisite.md)]

You also need:

* A Managed HSM in your subscription. See [Quickstart: Provision and activate a managed HSM using the Azure portal](quick-create-portal.md) to provision and activate a Managed HSM.
* To use Azure CLI: Azure CLI version 2.25.0 or later. Run `az --version` to find the version. If you need to install or upgrade, see [Install the Azure CLI](/cli/azure/install-azure-cli).
* To use Azure PowerShell: The [Azure PowerShell module](/powershell/azure/install-azure-powershell).

## Create a new role assignment

# [Azure portal](#tab/azure-portal)

<!-- TODO: Fill in portal steps for role assignment once portal experience is confirmed with Jack Richins. -->

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, select **Local RBAC**.

1. Select **Add role assignment**, choose the role, scope, and principal, then save.

# [Azure CLI](#tab/azure-cli)

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

# [Azure PowerShell](#tab/azure-powershell)

### Assign roles for all keys

Use the `New-AzKeyVaultRoleAssignment` cmdlet to assign a **Managed HSM Crypto User** role to a user for all **keys** (scope `/keys`).

```azurepowershell-interactive
New-AzKeyVaultRoleAssignment -HsmName <hsm-name> -RoleDefinitionName "Managed HSM Crypto User" -SignInName <user-principal-name> -Scope /keys
```

### Assign role for a specific key

```azurepowershell-interactive
New-AzKeyVaultRoleAssignment -HsmName <hsm-name> -RoleDefinitionName "Managed HSM Crypto User" -SignInName <user-principal-name> -Scope /keys/<key-name>
```

---

## List existing role assignments

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Local RBAC**.

   The portal displays all role assignments for the Managed HSM. You can filter by principal or scope.

# [Azure CLI](#tab/azure-cli)

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

# [Azure PowerShell](#tab/azure-powershell)

List all role assignments for the Managed HSM:

```azurepowershell-interactive
Get-AzKeyVaultRoleAssignment -HsmName <hsm-name>
```

List all role assignments for a specific user:

```azurepowershell-interactive
Get-AzKeyVaultRoleAssignment -HsmName <hsm-name> -SignInName <user-principal-name>
```

List role assignments for a specific user at a specific key scope:

```azurepowershell-interactive
Get-AzKeyVaultRoleAssignment -HsmName <hsm-name> -SignInName <user-principal-name> -Scope /keys/<key-name>
```

---

## Delete a role assignment

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Local RBAC**.

1. Locate the role assignment you want to remove.

1. Select the **Delete** (trash can) icon next to the assignment.

1. Confirm the deletion when prompted.

# [Azure CLI](#tab/azure-cli)

Use `az keyvault role assignment delete` command to delete a **Managed HSM Crypto Officer** role assigned to user `<user-principal-name>` for key `<key-name>`.

```azurecli-interactive
az keyvault role assignment delete --hsm-name <hsm-name> --role "Managed HSM Crypto Officer" --assignee <user-principal-name>  --scope /keys/<key-name>
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Remove-AzKeyVaultRoleAssignment` cmdlet to delete a role assignment:

```azurepowershell-interactive
Remove-AzKeyVaultRoleAssignment -HsmName <hsm-name> -RoleDefinitionName "Managed HSM Crypto Officer" -SignInName <user-principal-name> -Scope /keys/<key-name>
```

---

## List all available role definitions

# [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), navigate to your Managed HSM resource.

1. In the left menu, under **Settings**, select **Local RBAC**.

1. Select the **Roles** tab to view all available built-in and custom role definitions.

# [Azure CLI](#tab/azure-cli)

Use `az keyvault role definition list` command to list all the role definitions.

```azurecli-interactive
az keyvault role definition list --hsm-name <hsm-name>
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Get-AzKeyVaultRoleDefinition` cmdlet to list all role definitions:

```azurepowershell-interactive
Get-AzKeyVaultRoleDefinition -HsmName <hsm-name>
```

---

## Create a new role definition

> [!NOTE]
> Custom role definitions can only be managed by using Azure CLI or Azure PowerShell.

Managed HSM has several built-in (pre-defined) roles that are useful for most common usage scenarios. You can define your own role with a list of specific actions that the role is allowed to perform. Then you can assign this role to principals to grant them the permission to the specified actions. 

# [Azure CLI](#tab/azure-cli)

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

# [Azure PowerShell](#tab/azure-powershell)

Use the `New-AzKeyVaultRoleDefinition` cmdlet to create a custom role from a JSON file:

```azurepowershell-interactive
New-AzKeyVaultRoleDefinition -HsmName <hsm-name> -InputFile ./my-custom-role-definition.json
```

---

## Show details of a role definition

# [Azure CLI](#tab/azure-cli)

Use `az keyvault role definition show` command to see details of a specific role definition using name (a GUID).

```azurecli-interactive
az keyvault role definition show --hsm-name <hsm-name> --name xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Get-AzKeyVaultRoleDefinition` cmdlet to view role definitions. To view a specific custom role by name:

```azurepowershell-interactive
Get-AzKeyVaultRoleDefinition -HsmName <hsm-name> -RoleDefinitionName "My Custom Role"
```

---

## Update a custom role definition

# [Azure CLI](#tab/azure-cli)

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

# [Azure PowerShell](#tab/azure-powershell)

To update a custom role, modify the role object and re-create it:

```azurepowershell-interactive
$role = Get-AzKeyVaultRoleDefinition -HsmName <hsm-name> -RoleDefinitionName "My Custom Role"
$role.Permissions[0].DataActions = @("Microsoft.KeyVault/managedHsm/keys/read/action", "Microsoft.KeyVault/managedHsm/keys/write/action", "Microsoft.KeyVault/managedHsm/keys/backup/action", "Microsoft.KeyVault/managedHsm/keys/create")
New-AzKeyVaultRoleDefinition -HsmName <hsm-name> -Role $role
```

---

## Delete custom role definition

# [Azure CLI](#tab/azure-cli)

Use the Azure CLI [az keyvault role definition delete](/cli/azure/keyvault/role/definition#az-keyvault-role-definition-delete) command to delete a custom role definition using name (a GUID).

```azurecli-interactive
az keyvault role definition delete --hsm-name <hsm-name> --name xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
```

# [Azure PowerShell](#tab/azure-powershell)

Use the `Remove-AzKeyVaultRoleDefinition` cmdlet to delete a custom role definition:

```azurepowershell-interactive
Remove-AzKeyVaultRoleDefinition -HsmName <hsm-name> -RoleName "My Custom Role"
```

---

> [!NOTE]
> Built-in roles cannot be deleted. When custom roles are deleted, all the role assignments using that custom role become defunct.

## Next steps

- See an overview of [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview).
- Learn more about [Managed HSM access control model](access-control.md)
- See all the [built-in roles for Managed HSM local RBAC](built-in-roles.md)
