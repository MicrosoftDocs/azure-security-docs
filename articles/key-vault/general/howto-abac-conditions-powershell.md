---
title: Add role assignment conditions for Key Vault secrets by using Azure PowerShell (preview)
description: Learn how to add an Azure ABAC condition to a Key Vault role assignment by using Azure PowerShell to restrict secret access by name.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 07/16/2026
ai-usage: ai-assisted
---

# Add role assignment conditions for Key Vault secrets by using Azure PowerShell (preview)

> [!NOTE]
> Azure ABAC for Key Vault is in preview. Some aspects might change before general availability. For preview terms, see the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

In most cases, an Azure role assignment grants the permissions you need to Azure resources. In some cases, you might want more granular access control by adding a role assignment *condition*.

This article shows how to use Azure PowerShell to add an Azure attribute-based access control (Azure ABAC) condition to a Key Vault role assignment so that a principal can only read secrets whose names start with a specific prefix.

For the full set of supported actions and attributes, see [Actions and attributes for Azure Key Vault ABAC conditions (preview)](rbac-abac.md).

> [!IMPORTANT]
> ABAC conditions only work when the key vault's permission model is set to **Azure role-based access control**. Legacy vault access policies don't support conditions.

## Prerequisites

- For information about the prerequisites to add or edit role assignment conditions, see [Conditions prerequisites](/azure/role-based-access-control/conditions-prerequisites).
- The target key vault must have its permission model set to **Azure role-based access control**.
- Azure PowerShell installed locally, or use Azure Cloud Shell.

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

## Step 1: Install prerequisites

1. Open a PowerShell window.

1. Use [Get-InstalledModule](/powershell/module/powershellget/get-installedmodule) to check the versions of installed modules.

   ```azurepowershell-interactive
   Get-InstalledModule -Name Az
   Get-InstalledModule -Name Az.Resources
   Get-InstalledModule -Name Az.KeyVault
   Get-InstalledModule -Name Microsoft.Graph.Users
   ```

1. If necessary, use [Install-Module](/powershell/module/powershellget/install-module) to install the required versions of the `Az`, `Az.Resources`, and `Az.KeyVault` modules. You only need the `Microsoft.Graph.Users` module if you plan to create a new user with `New-MgUser` in [Step 3](#step-3-create-a-user); you can skip it if you reuse an existing user.

   ```azurepowershell-interactive
   Install-Module -Name Az
   Install-Module -Name Az.Resources
   Install-Module -Name Az.KeyVault
   Install-Module -Name Microsoft.Graph.Users
   ```

1. Close and reopen PowerShell to refresh the session.

## Step 2: Sign in to Azure

1. Use [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) and follow the instructions to sign in as a [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator) or [Owner](/azure/role-based-access-control/built-in-roles#owner).

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

1. Use [Get-AzSubscription](/powershell/module/az.accounts/get-azsubscription) to list your subscriptions. Format the output as a list so the full subscription IDs aren't truncated.

   ```azurepowershell-interactive
   Get-AzSubscription | Format-List Name, Id, TenantId
   ```

   Alternatively, display the results in a table with an untruncated `Id` column:

   ```azurepowershell-interactive
   Get-AzSubscription | Format-Table Name, Id, State -AutoSize
   ```

1. Set your subscription as the active subscription.

   ```azurepowershell-interactive
   $subscriptionId = "<subscriptionId>"
   $context = Get-AzSubscription -SubscriptionId $subscriptionId
   Set-AzContext $context
   ```

## Step 3: Create a user

Use [New-MgUser](/powershell/module/microsoft.graph.users/new-mguser) to create a user, or find an existing user. This article uses *User1* as the example.

> [!NOTE]
> `New-MgUser` belongs to the `Microsoft.Graph.Users` module (installed in Step 1), not to the `Az` modules. It also requires a separate Microsoft Graph sign-in from `Connect-AzAccount`. Before you run it, connect with the `User.ReadWrite.All` scope:
>
> ```azurepowershell-interactive
> Connect-MgGraph -Scopes "User.ReadWrite.All"
> ```
>
> If you're reusing an existing user, you can skip both the `Microsoft.Graph.Users` module and `Connect-MgGraph` and supply that user's object ID.

Initialize a variable with the user's object ID.

```azurepowershell-interactive
$userObjectId = "<userObjectId>"
```

## Step 4: Set up the key vault

1. Use [New-AzKeyVault](/powershell/module/az.keyvault/new-azkeyvault) to create a key vault that uses Azure RBAC as its permission model.

   ```azurepowershell-interactive
   New-AzKeyVault -Name "<keyVaultName>" -ResourceGroupName "<resourceGroup>" -Location "<location>"
   ```

   > [!NOTE]
   > In `Az.KeyVault` 6.0.0 and later, Azure RBAC is the default permission model, so you don't need to specify an extra switch. The `-EnableRbacAuthorization` switch was *removed* in 6.0.0. If you include it, the command now fails with `A parameter cannot be found that matches parameter name 'EnableRbacAuthorization'`. Use `-DisableRbacAuthorization` only if you explicitly want the legacy vault access policy model instead.
   >
   > If you're running an older `Az.KeyVault` version (earlier than 6.0.0), RBAC isn't the default, so you must add the `-EnableRbacAuthorization` switch:
   >
   > ```azurepowershell-interactive
   > New-AzKeyVault -Name "<keyVaultName>" -ResourceGroupName "<resourceGroup>" -Location "<location>" -EnableRbacAuthorization
   > ```
   >
   > Check your installed version with `Get-InstalledModule -Name Az.KeyVault`.

1. Grant yourself a Key Vault *data-plane* role so you can create secrets. When a vault uses Azure RBAC (the default), control-plane roles such as **Owner** or **Contributor** don't grant access to secret values. Without a data-plane role, `Set-AzKeyVaultSecret` fails with `Operation returned an invalid status code 'Forbidden'` and `Assignment: (not found)`.

   ```azurepowershell-interactive
   # Object ID of the signed-in user (use -UserPrincipalName "you@tenant.com" if this returns nothing)
   $me = (Get-AzADUser -SignedIn).Id

   # Scope the assignment to the vault you just created
   $vaultScope = "/subscriptions/$subscriptionId/resourceGroups/<resourceGroup>/providers/Microsoft.KeyVault/vaults/<keyVaultName>"

   New-AzRoleAssignment -ObjectId $me `
     -RoleDefinitionName "Key Vault Secrets Officer" `
     -Scope $vaultScope
   ```

   > [!NOTE]
   > Use **Key Vault Secrets Officer** to create and manage secret values, or **Key Vault Administrator** for full data-plane access. **Key Vault Secrets User** is read-only. After you assign a role, wait 1–5 minutes for propagation before you run the next command. A `Forbidden` error immediately after assignment usually means the role hasn't propagated yet.

1. Use [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret) to create a secret named `test-app-secret1`.

   ```azurepowershell-interactive
   $secretValue = ConvertTo-SecureString "<secretValue>" -AsPlainText -Force
   Set-AzKeyVaultSecret -VaultName "<keyVaultName>" -Name "test-app-secret1" -SecretValue $secretValue
   ```

   > [!NOTE]
   > The secret *name* is the resource attribute evaluated by the condition. Choose names that reflect the prefix you plan to restrict access to.

1. Create a second secret named `test-db-password`.

   ```azurepowershell-interactive
   Set-AzKeyVaultSecret -VaultName "<keyVaultName>" -Name "test-db-password" -SecretValue $secretValue
   ```

1. Initialize variables with the names you used.

   ```azurepowershell-interactive
   $resourceGroup = "<resourceGroup>"
   $keyVaultName = "<keyVaultName>"
   $secretNameAllowed = "test-app-secret1"
   $secretNameDenied = "test-db-password"
   ```

## Step 5: Assign a role with a condition

1. Initialize the **Key Vault Secrets User** role variables.

   ```azurepowershell-interactive
   $roleDefinitionName = "Key Vault Secrets User"
   $roleDefinitionId = "4633458b-17de-408a-b874-0445c86b69e6"
   ```

1. Initialize the scope for the resource group. If you didn't set the `$resourceGroup` variable earlier in this session, set it to the name of the resource group that contains your key vault.

   ```azurepowershell-interactive
   $resourceGroup = "<resourceGroup>"
   $scope = "/subscriptions/$subscriptionId/resourceGroups/$resourceGroup"
   ```

1. Initialize the condition.

   ```azurepowershell-interactive
   $condition = "((!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})) OR (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'))"
   ```

   > [!IMPORTANT]
   > The condition must be a single-line string. Multi-line conditions are rejected. If you build the condition in a here-string or across multiple lines, flatten it first:
   >
   > ```azurepowershell-interactive
   > $condition = $condition -replace '\s+', ' '
   > ```
   >
   > In PowerShell, if your condition includes a dollar sign (`$`), prefix it with a backtick (`` ` ``).

1. Initialize the condition version and description.

   ```azurepowershell-interactive
   $conditionVersion = "2.0"
   $description = "Read access to secrets whose names start with test-app"
   ```

1. Use [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) to assign the **Key Vault Secrets User** role with a condition to the user at resource group scope.

   ```azurepowershell-interactive
   New-AzRoleAssignment -ObjectId $userObjectId -Scope $scope -RoleDefinitionId $roleDefinitionId -Description $description -Condition $condition -ConditionVersion $conditionVersion
   ```

   Example output:

   ```output
   RoleAssignmentId   : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>/providers/Microsoft.Authorization/roleAssignments/<roleAssignmentId>
   Scope              : /subscriptions/<subscriptionId>/resourceGroups/<resourceGroup>
   DisplayName        : User1
   SignInName         : user1@contoso.com
   RoleDefinitionName : Key Vault Secrets User
   RoleDefinitionId   : 4633458b-17de-408a-b874-0445c86b69e6
   ObjectId           : <userObjectId>
   ObjectType         : User
   CanDelegate        : False
   Description        : Read access to secrets whose names start with test-app
   ConditionVersion   : 2.0
   Condition          : ((!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})) OR (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'))
   ```

## Step 6: Test the condition

1. Open a new PowerShell window.

1. Use [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) to sign in as User1.

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

1. Initialize the variables you used earlier.

   ```azurepowershell-interactive
   $keyVaultName = "<keyVaultName>"
   $secretNameAllowed = "test-app-secret1"
   $secretNameDenied = "test-db-password"
   ```

1. Use [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret) to try to read the denied secret.

   ```azurepowershell-interactive
   Get-AzKeyVaultSecret -VaultName $keyVaultName -Name $secretNameDenied -AsPlainText
   ```

   Example output. Notice that the read fails because of the condition:

   ```output
   Get-AzKeyVaultSecret : Operation returned an invalid status code 'Forbidden'
   Caller is not authorized to perform action on resource.
   If role assignments, deny assignments or role definitions changed
   recently, please observe propagation time.
   ...
   ForbiddenByRbac
   ```

1. Read the secret whose name starts with `test-app`.

   ```azurepowershell-interactive
   Get-AzKeyVaultSecret -VaultName $keyVaultName -Name $secretNameAllowed -AsPlainText
   ```

   You can read this secret because its name starts with `test-app`.

## Step 7: (Optional) Edit the condition

1. In the other PowerShell window, use [Get-AzRoleAssignment](/powershell/module/az.resources/get-azroleassignment) to get the role assignment you added.

   ```azurepowershell-interactive
   $testRa = Get-AzRoleAssignment -Scope $scope -RoleDefinitionName $roleDefinitionName -ObjectId $userObjectId | Where-Object { $_.Scope -eq $scope }
   ```

   > [!IMPORTANT]
   > `Get-AzRoleAssignment` also returns assignments *inherited* from higher scopes (for example, the same role assigned at the subscription level). If the principal holds this role at more than one scope, `$testRa` becomes an array, and the next step fails with `The property 'Condition' cannot be found on this object`. The `Where-Object { $_.Scope -eq $scope }` filter keeps only the assignment at your resource group scope so `$testRa` is a single object.

1. Edit the condition to also allow secrets whose names start with `api-`.

   ```azurepowershell-interactive
   $condition = "((!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})) OR (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app' OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'api-'))"
   ```

   For the `Get secret` action, this condition allows reading a secret only if its name starts with `test-app` or `api-`. All other actions pass through the condition normally.

1. Update the condition and description on the assignment.

   ```azurepowershell-interactive
   $testRa.Condition = $condition
   $testRa.Description = "Read access to secrets whose names start with test-app or api-"
   ```

1. Use [Set-AzRoleAssignment](/powershell/module/az.resources/set-azroleassignment) to save the change.

   ```azurepowershell-interactive
   Set-AzRoleAssignment -InputObject $testRa -PassThru
   ```

## Step 8: Clean up resources

1. Use [Remove-AzRoleAssignment](/powershell/module/az.resources/remove-azroleassignment) to remove the role assignment and condition.

   ```azurepowershell-interactive
   Remove-AzRoleAssignment -ObjectId $userObjectId -RoleDefinitionName $roleDefinitionName -ResourceGroupName $resourceGroup
   ```

1. Delete the key vault you created.
1. Delete the user you created.

## Related content

- [Actions and attributes for Azure Key Vault ABAC conditions (preview)](rbac-abac.md)
- [Add role assignment conditions for Key Vault secrets by using Azure CLI (preview)](howto-abac-conditions-cli.md)
- [Troubleshoot Azure role assignment conditions](/azure/role-based-access-control/conditions-troubleshoot)
- [Troubleshoot access issues for Azure Key Vault](troubleshooting-access-issues.md)
