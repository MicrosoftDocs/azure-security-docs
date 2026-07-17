---
title: Add role assignment conditions for Key Vault secrets by using Azure CLI (preview)
description: Learn how to add an Azure ABAC condition to a Key Vault role assignment by using Azure CLI to restrict secret access by name.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 07/16/2026
ai-usage: ai-assisted
---

# Add role assignment conditions for Key Vault secrets by using Azure CLI (preview)

> [!NOTE]
> Azure ABAC for Key Vault is in preview. Some aspects might change before general availability. For preview terms, see the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

In most cases, an Azure role assignment grants the permissions you need to Azure resources. In some cases, you might want more granular access control by adding a role assignment *condition*.

This article shows how to use Azure CLI to add an Azure attribute-based access control (Azure ABAC) condition to a Key Vault role assignment so that a principal can only read secrets whose names start with a specific prefix.

For the full set of supported actions and attributes, see [Actions and attributes for Azure Key Vault ABAC conditions (preview)](rbac-abac.md).

> [!IMPORTANT]
> ABAC conditions only work when the key vault's permission model is set to **Azure role-based access control**. Legacy vault access policies don't support conditions.

## Prerequisites

- For information about the prerequisites to add or edit role assignment conditions, see [Conditions prerequisites](/azure/role-based-access-control/conditions-prerequisites).
- The target key vault must have its permission model set to **Azure role-based access control**.
- Azure CLI installed locally, or use Azure Cloud Shell.

## Condition

In this article, you restrict access to secrets whose names begin with `test-app`. If the user tries to read a secret without the `test-app` name prefix, access isn't allowed.

Here's what the condition looks like:

```
(
  (
    !(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})
  )
  OR
  (
    @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'
  )
)
```

## Step 1: Sign in to Azure

1. Use [az login](/cli/azure/reference-index#az-login) and follow the instructions to sign in as a [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) or [Owner](/azure/role-based-access-control/built-in-roles#owner).

   ```azurecli-interactive
   az login
   ```

1. Use [az account show](/cli/azure/account#az-account-show) to get your subscription ID.

   ```azurecli-interactive
   az account show
   ```

1. Initialize a variable with your subscription ID.

   ```azurecli-interactive
   subscriptionId="<subscriptionId>"
   ```

## Step 2: Create a user

1. Use [az ad user create](/cli/azure/ad/user#az-ad-user-create) to create a user, or find an existing user. This article uses *User1* as the example.

1. Initialize a variable with the user's object ID.

   ```azurecli-interactive
   userObjectId="<userObjectId>"
   ```

## Step 3: Set up the key vault

> [!NOTE]
> The commands in this article use **Bash** syntax (`variable="value"` and `$variable`). If you run them from **Windows PowerShell** or **PowerShell 7**, use PowerShell variable syntax instead (`$variable = "value"`) and ignore the Bash history-expansion note in [Step 4](#step-4-assign-a-role-with-a-condition). The `az` commands themselves are identical in both shells.

1. Use [az keyvault create](/cli/azure/keyvault#az-keyvault-create) to create a key vault that uses Azure RBAC as its permission model.

   ```azurecli-interactive
   az keyvault create --name "<keyVaultName>" --resource-group "<resourceGroup>" --location "<location>" --enable-rbac-authorization true
   ```

1. Grant yourself a Key Vault *data-plane* role so you can create secrets. When a vault uses Azure RBAC, control-plane roles such as **Owner** or **Contributor** don't grant access to secret values. Without a data-plane role, `az keyvault secret set` fails with a `(Forbidden) Caller is not authorized to perform action on resource` error for the `Microsoft.KeyVault/vaults/secrets/setSecret/action` action.

   ```azurecli-interactive
   myObjectId=$(az ad signed-in-user show --query id -o tsv)
   vaultScope="/subscriptions/$subscriptionId/resourceGroups/<resourceGroup>/providers/Microsoft.KeyVault/vaults/<keyVaultName>"
   az role assignment create --assignee-object-id $myObjectId --assignee-principal-type User --role "Key Vault Secrets Officer" --scope $vaultScope
   ```

   > [!NOTE]
   > Use **Key Vault Secrets Officer** to create and manage secret values, or **Key Vault Administrator** for full data-plane access. **Key Vault Secrets User** is read-only. After you assign a role, wait 1–5 minutes for propagation before you run the next command. A `Forbidden` error immediately after assignment usually means the role hasn't propagated yet.

1. Use [az keyvault secret set](/cli/azure/keyvault/secret#az-keyvault-secret-set) to create a secret named `test-app-secret1`.

   ```azurecli-interactive
   az keyvault secret set --vault-name "<keyVaultName>" --name "test-app-secret1" --value "<secretValue>"
   ```

   > [!NOTE]
   > The secret *name* is the resource attribute evaluated by the condition. Choose names that reflect the prefix you plan to restrict access to.

1. Create a second secret named `test-db-password`.

   ```azurecli-interactive
   az keyvault secret set --vault-name "<keyVaultName>" --name "test-db-password" --value "<secretValue>"
   ```

1. Initialize variables with the names you used.

   ```azurecli-interactive
   resourceGroup="<resourceGroup>"
   keyVaultName="<keyVaultName>"
   secretNameAllowed="test-app-secret1"
   secretNameDenied="test-db-password"
   ```

## Step 4: Assign a role with a condition

1. Initialize the **Key Vault Secrets User** role variables.

   ```azurecli-interactive
   roleDefinitionName="Key Vault Secrets User"
   roleDefinitionId="4633458b-17de-408a-b874-0445c86b69e6"
   ```

1. Initialize the scope for the resource group.

   ```azurecli-interactive
   scope="/subscriptions/$subscriptionId/resourceGroups/$resourceGroup"
   ```

1. Initialize the condition.

   ```azurecli-interactive
   condition="((!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})) OR (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'))"
   ```

   For the `Get secret` action, this condition allows reading a secret only if its name starts with `test-app`. All other actions (non-`getSecret`) are unaffected.

   > [!NOTE]
   > The condition must be a single-line string. Multi-line conditions are rejected. If you build the condition across multiple lines, flatten it first. For example:
   >
   > ```bash
   > condition=$(echo "$condition" | tr -s ' \n' ' ')
   > ```
   >
   > In Bash, if history expansion is enabled, you might see `bash: !: event not found` because of the exclamation point (`!`) in the condition. Disable history expansion with `set +H`, then re-enable it later with `set -H`.

1. Initialize the condition version and description.

   ```azurecli-interactive
   conditionVersion="2.0"
   description="Read access to secrets whose names start with test-app"
   ```

1. Use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) to assign the **Key Vault Secrets User** role with a condition to the user at resource group scope.

   ```azurecli-interactive
   az role assignment create --assignee-object-id $userObjectId --scope $scope --role $roleDefinitionId --description "$description" --condition "$condition" --condition-version $conditionVersion
   ```

   Example output:

   ```json
   {
     "canDelegate": null,
     "condition": "((!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})) OR (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'))",
     "conditionVersion": "2.0",
     "description": "Read access to secrets whose names start with test-app",
     "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
     "name": "{roleAssignmentId}",
     "principalId": "{userObjectId}",
     "principalType": "User",
     "resourceGroup": "{resourceGroup}",
     "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/4633458b-17de-408a-b874-0445c86b69e6",
     "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
     "type": "Microsoft.Authorization/roleAssignments"
   }
   ```

## Step 5: Test the condition

1. Open a new command window.

1. Use [az login](/cli/azure/reference-index#az-login) to sign in as User1.

   ```azurecli-interactive
   az login
   ```

1. Initialize the variables you used earlier.

   ```azurecli-interactive
   keyVaultName="<keyVaultName>"
   secretNameAllowed="test-app-secret1"
   secretNameDenied="test-db-password"
   ```

1. Use [az keyvault secret show](/cli/azure/keyvault/secret#az-keyvault-secret-show) to try to read the denied secret.

   ```azurecli-interactive
   az keyvault secret show --vault-name $keyVaultName --name $secretNameDenied
   ```

   Example output. Notice that the read fails because of the condition:

   ```output
   (Forbidden) Caller is not authorized to perform action on resource.
   If role assignments, deny assignments or role definitions changed
   recently, please observe propagation time.
   ...
   Code: ForbiddenByRbac
   ```

1. Read the secret whose name starts with `test-app`.

   ```azurecli-interactive
   az keyvault secret show --vault-name $keyVaultName --name $secretNameAllowed
   ```

   Example output. Notice that you can read the secret because its name starts with `test-app`:

   ```json
   {
     "attributes": { },
     "contentType": null,
     "id": "https://<keyVaultName>.vault.azure.net/secrets/test-app-secret1/<version>",
     "name": "test-app-secret1",
     "tags": {},
     "value": "<secretValue>"
   }
   ```

## Step 6: (Optional) Edit the condition

1. In the original command window, use [az role assignment list](/cli/azure/role/assignment#az-role-assignment-list) to get the role assignment you added.

   ```azurecli-interactive
   az role assignment list --assignee $userObjectId --resource-group $resourceGroup
   ```

1. Create a JSON file with the following format and update the `condition` and `description` properties to also allow secrets whose names start with `api-`.

   ```json
   {
     "canDelegate": null,
     "condition": "((!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})) OR (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app' OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'api-'))",
     "conditionVersion": "2.0",
     "description": "Read access to secrets whose names start with test-app or api-",
     "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
     "name": "{roleAssignmentId}",
     "principalId": "{userObjectId}",
     "principalName": "user1@contoso.com",
     "principalType": "User",
     "resourceGroup": "{resourceGroup}",
     "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/4633458b-17de-408a-b874-0445c86b69e6",
     "roleDefinitionName": "Key Vault Secrets User",
     "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}",
     "type": "Microsoft.Authorization/roleAssignments"
   }
   ```

1. Use [az role assignment update](/cli/azure/role/assignment#az-role-assignment-update) to update the condition.

   ```azurecli-interactive
   az role assignment update --role-assignment "./path/roleassignment.json"
   ```

## Step 7: Clean up resources

1. Use [az role assignment delete](/cli/azure/role/assignment#az-role-assignment-delete) to remove the role assignment and condition.

   ```azurecli-interactive
   az role assignment delete --assignee $userObjectId --role "$roleDefinitionName" --resource-group $resourceGroup
   ```

1. Delete the key vault you created.
1. Delete the user you created.

## Related content

- [Actions and attributes for Azure Key Vault ABAC conditions (preview)](rbac-abac.md)
- [Add role assignment conditions for Key Vault secrets by using Azure PowerShell (preview)](howto-abac-conditions-powershell.md)
- [Troubleshoot Azure role assignment conditions](/azure/role-based-access-control/conditions-troubleshoot)
- [Troubleshoot access issues for Azure Key Vault](troubleshooting-access-issues.md)
