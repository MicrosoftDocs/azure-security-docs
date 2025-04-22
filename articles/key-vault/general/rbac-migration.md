---
title: Migrate to Azure role-based access control | Microsoft Docs
description: Learn how to migrate from vault access policies to Azure roles.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/17/2025
ms.author: mbaldwin

---
# Migrate from vault access policy to an Azure role-based access control permission model

Azure Key Vault offers two authorization systems: Azure role-based access control (Azure RBAC), and an access policy model. Azure RBAC is the default and recommended authorization system for Azure Key Vault. For a comparison of the two methods of authorization, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md).

This article provides the information necessary to migrate a key vault from an access policy model to an Azure RBAC model.

## Access policies to Azure roles mapping

Azure RBAC has several Azure built-in roles that you can assign to users, groups, service principals, and managed identities. If the built-in roles don't meet the specific needs of your organization, you can create your own [Azure custom roles](/azure/role-based-access-control/custom-roles).

[Key Vault built-in roles](rbac-guide.md#azure-built-in-roles-for-key-vault-data-plane-operations) for keys, certificates, and secrets access management:
- Key Vault Administrator
- Key Vault Reader
- Key Vault Purge Operator
- Key Vault Certificates Officer
- Key Vault Certificate User
- Key Vault Crypto Officer
- Key Vault Crypto User
- Key Vault Crypto Service Encryption User
- Key Vault Crypto Service Release User
- Key Vault Secrets Officer
- Key Vault Secrets User

For more information about existing built-in roles, see [Azure built-in roles](/azure/role-based-access-control/built-in-roles)

Vault access policies can be assigned with individually selected permissions or with predefined permission templates.

Access policy predefined permission templates:
- Key, Secret, Certificate Management
- Key & Secret Management
- Secret & Certificate Management
- Key Management
- Secret Management
- Certificate Management
- SQL Server Connector
- Azure Data Lake Storage or Azure Storage
- Azure Backup
- Exchange Online Customer Key
- SharePoint Online Customer Key
- Azure Information BYOK

### Access policy templates to Azure roles mapping
| Access policy template | Operations | Azure role |
| --- | --- | --- |
| Key, Secret, Certificate Management | Keys: all operations <br>Certificates: all operations<br>Secrets: all operations | Key Vault Administrator |
| Key & Secret Management | Keys: all operations <br>Secrets: all operations| Key Vault Crypto Officer <br> Key Vault Secrets Officer |
| Secret & Certificate Management | Certificates: all operations <br>Secrets: all operations| Key Vault Certificates Officer <br> Key Vault Secrets Officer|
| Key Management | Keys: all operations| Key Vault Crypto Officer|
| Secret Management | Secrets: all operations| Key Vault Secrets Officer|
| Certificate Management | Certificates: all operations | Key Vault Certificates Officer|
| SQL Server Connector | Keys: get, list, wrap key, unwrap key | Key Vault Crypto Service Encryption User|
| Azure Data Lake Storage or Azure Storage | Keys: get, list,  unwrap key | N/A<br> Custom role required|
| Azure Backup | Keys: get, list, backup<br> Secrets: get, list, backup | N/A<br> Custom role required|
| Exchange Online Customer Key | Keys: get, list, wrap key, unwrap key | Key Vault Crypto Service Encryption User|
| Exchange Online Customer Key | Keys: get, list, wrap key, unwrap key | Key Vault Crypto Service Encryption User|
| Azure Information BYOK | Keys: get, decrypt, sign | N/A<br>Custom role required|

## Assignment scopes mapping  

Azure RBAC for Key Vault allows roles assignment at following scopes:
- Management group
- Subscription
- Resource group
- Key Vault resource
- Individual key, secret, and certificate

The vault access policy permission model is limited to assigning policies only at Key Vault resource level.

In general, it's best practice to have one key vault per application and manage access at key vault level. There are scenarios when managing access at other scopes can simplify access management.

- **Infrastructure, security administrators and operators**: managing group of key vaults at management group, subscription or resource group level with vault access policies requires maintaining policies for each key vault. Azure RBAC allows creating one role assignment at management group, subscription, or resource group. That assignment will apply to any new key vaults created under the same scope. In this scenario, it's recommended to use Privileged Identity Management with just-in time access over providing permanent access.
 
- **Applications**: there are scenarios when application would need to share secret with other application. Using vault access polices separate key vault had to be created to avoid giving access to all secrets. Azure RBAC allows assign role with scope for individual secret instead using single key vault.

## Vault access policy to Azure RBAC migration steps

There are many differences between Azure RBAC and vault access policy permission model. In order, to avoid outages during migration, below steps are recommended.
 
1. **Identify and assign roles**: identify built-in roles based on mapping table above and create custom roles when needed. Assign roles at scopes, based on scopes mapping guidance. For more information on how to assign roles to key vault, see [Provide access to Key Vault with an Azure role-based access control](rbac-guide.md)
1. **Validate roles assignment**: role assignments in Azure RBAC can take several minutes to propagate. For guide how to check role assignments, see [List roles assignments at scope](/azure/role-based-access-control/role-assignments-list-portal#list-role-assignments-for-a-user-at-a-scope)
1. **Configure monitoring and alerting on key vault**: it's important to enable logging and setup alerting for access denied exceptions. For more information, see [Monitoring and alerting for Azure Key Vault](./alert.md)
1. **Set Azure role-based access control permission model on Key Vault**: enabling Azure RBAC permission model will invalidate all existing access policies. If an error, permission model can be switched back with all existing access policies remaining untouched.

## Prerequisites

Before starting the migration, ensure you have:

1. **Required permissions**: You must have the following permissions on the key vault:
   - `Microsoft.Authorization/roleAssignments/write` permission, included in Owner and User Access Administrator roles
   - `Microsoft.KeyVault/vaults/write` permission, included in the Key Vault Contributor role
   
   > [!NOTE]
   > Classic subscription administrator roles (Service Administrator and Co-Administrator) are not supported.
1. **Inventory of applications and identities**:  List all applications, services, and users that access the key vault, and document all current access policies and the permissions they grant.
1. **Monitoring configured**: Enable the diagnostic settings on the key vault to detect any access issues during migration. Set up alerts for access denied errors (HTTP 401/403).

## How to migrate

Follow these steps to migrate your key vault to RBAC from access policies:

### Inventory current access policies

Document all existing access policies, noting the security principals (users, groups, service principals) and their permissions.

# [Azure CLI](#tab/cli)
Use the Azure CLI [az keyvault show](/cli/azure/keyvault#az-keyvault-show) command to retrieve the access policies:

```azurecli
# List all current access policies
az keyvault show --name <vault-name> --resource-group <resource-group-name> --query properties.accessPolicies
```

# [Azure PowerShell](#tab/powershell)
Use the [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault) cmdlet to retrieve the access policies:

```powershell
# List all current access policies
$vault = Get-AzKeyVault -VaultName "<vault-name>" -ResourceGroupName "<resource-group-name>"
$vault.AccessPolicies
```

# [Azure portal](#tab/portal)
In the Azure portal:

1. Navigate to your key vault
2. Select **Access policies** under Settings
3. Document all existing access policies, noting:
    - Identity (user, group, or service principal)
    - Key, Secret, and Certificate permissions granted

---

### Create equivalent RBAC role assignments

For each security principal with an access policy, create one or more RBAC role assignments based on the mapping table above.

# [Azure CLI](#tab/cli)
Use the [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) command to grant appropriate roles:

```azurecli
# Example for Key Vault Administrator role:
az role assignment create --role "Key Vault Administrator" --assignee "<object-id-or-email>" --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Example for Key Vault Secrets Officer:
az role assignment create --role "Key Vault Secrets Officer" --assignee "<object-id-or-email>" --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Example for Key Vault Crypto Officer:
az role assignment create --role "Key Vault Crypto Officer" --assignee "<object-id-or-email>" --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Example for Key Vault Certificates Officer:
az role assignment create --role "Key Vault Certificates Officer" --assignee "<object-id-or-email>" --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
```

# [Azure PowerShell](#tab/powershell)
Use the [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment) cmdlet to grant appropriate roles:

```powershell
# Example for Key Vault Administrator role:
New-AzRoleAssignment -RoleDefinitionName "Key Vault Administrator" -ObjectId "<object-id>" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Example for Key Vault Secrets Officer:
New-AzRoleAssignment -RoleDefinitionName "Key Vault Secrets Officer" -ObjectId "<object-id>" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Example for Key Vault Crypto Officer:
New-AzRoleAssignment -RoleDefinitionName "Key Vault Crypto Officer" -ObjectId "<object-id>" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"

# Example for Key Vault Certificates Officer:
New-AzRoleAssignment -RoleDefinitionName "Key Vault Certificates Officer" -ObjectId "<object-id>" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group-name>/providers/Microsoft.KeyVault/vaults/<vault-name>"
```

# [Azure portal](#tab/portal)
In the Azure portal:

1. Navigate to your key vault
2. Select **Access control (IAM)**
3. Click **Add** > **Add role assignment**
4. Select the appropriate role based on the access policy mapping
5. Search for and select the user, group, or service principal
6. Click **Review + assign** to create the role assignment
7. Repeat for each identity that needs access

---

### Enable RBAC permission model

After creating all necessary role assignments, switch the vault to use the RBAC permission model.

# [Azure CLI](#tab/cli)
Use the [az keyvault update](/cli/azure/keyvault#az-keyvault-update) command to enable RBAC:

```azurecli
# Switch the vault to RBAC permission model
az keyvault update --name <vault-name> --resource-group <resource-group-name> --enable-rbac-authorization true
```

# [Azure PowerShell](#tab/powershell)
Use the [Update-AzKeyVault](/powershell/module/az.keyvault/update-azkeyvault) cmdlet to enable RBAC:

```powershell
# Switch the vault to RBAC permission model
$vault = Get-AzKeyVault -VaultName "<vault-name>" -ResourceGroupName "<resource-group-name>"
Update-AzKeyVault -VaultName $vault.VaultName -ResourceGroupName $vault.ResourceGroupName -EnableRbacAuthorization $true
```

# [Azure portal](#tab/portal)
In the Azure portal:

1. Navigate to your key vault
2. Select **Properties** under Settings
3. Change **Permission model** to **Azure role-based access control**
4. Click **Save**
---

### Validate access

Test access to the vault to ensure all applications and users can still perform required operations.

# [Azure CLI](#tab/cli)
Test your access with these commands:

```azurecli
# Try to list secrets to verify access
az keyvault secret list --vault-name <vault-name>

# Try to get a secret to verify access
az keyvault secret show --vault-name <vault-name> --name <secret-name>
```

# [Azure PowerShell](#tab/powershell)
Test your access with these cmdlets:

```powershell
# Try to list secrets to verify access
Get-AzKeyVaultSecret -VaultName "<vault-name>"

# Try to get a secret to verify access
Get-AzKeyVaultSecret -VaultName "<vault-name>" -Name "<secret-name>"
```

# [Azure portal](#tab/portal)
In the Azure portal:

1. Try to access secrets, keys, or certificates based on your assigned roles
2. Verify that applications using the vault still function correctly

---

## Migration governance with Azure Policy

Using the Azure Policy service, you can govern RBAC permission model migration across your vaults. You can create a custom policy definition to audit existing key vaults and enforce all new key vaults to use the Azure RBAC permission model.

### Create and assign policy definition for Key Vault Azure RBAC permission model

1. Navigate to Policy resource
2. Select **Assignments** under **Authoring** on the left side of the Azure Policy page
3. Select **Assign policy** at the top of the page
4. Enter the following information:
    - Define the scope of the policy by choosing the subscription and resource group
    - Select the policy definition: "[[Preview]: Azure Key Vault should use RBAC permission model](https://portal.azure.com/#view/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F12d4fa5e-1f9f-4c21-97a9-b99b3c6611b5)"
    - Define the desired effect of the policy (Audit, Deny, or Disabled)
5. Complete the assignment by reviewing and creating it

Once the policy is assigned, it can take up to 24 hours to complete the scan. After the scan is completed, you can see compliance results in the Azure Policy dashboard.

## Monitoring and alerting

After migration, set up proper monitoring to detect any access issues:

# [Azure CLI](#tab/cli)
Use the [az monitor diagnostic-settings create](/cli/azure/monitor/diagnostic-settings#az-monitor-diagnostic-settings-create) command:

```azurecli
# Enable diagnostics logging for Key Vault
az monitor diagnostic-settings create --resource <vault-id> --name KeyVaultLogs --logs "[{\"category\":\"AuditEvent\",\"enabled\":true}]" --workspace <log-analytics-workspace-id>
```

# [Azure PowerShell](#tab/powershell)
Use the [Set-AzDiagnosticSetting](/powershell/module/az.monitor/set-azdiagnosticsetting) cmdlet:

```powershell
# Get the vault resource ID
$vaultResourceId = (Get-AzKeyVault -VaultName "<vault-name>" -ResourceGroupName "<resource-group-name>").ResourceId

# Enable diagnostics logging for Key Vault
$logs = @()
$logs += New-AzDiagnosticSettingLogSettingsObject -Category "AuditEvent" -Enabled $true
Set-AzDiagnosticSetting -ResourceId $vaultResourceId -Name "KeyVaultLogs" -WorkspaceId "<log-analytics-workspace-id>" -Log $logs
```

# [Azure portal](#tab/portal)
In the Azure portal:

1. Navigate to your key vault
2. Select **Diagnostic settings** under Monitoring
3. Click **Add diagnostic setting**
4. Enter a name for the setting (for example, "KeyVaultLogs")
5. Under "Logs", check the **AuditEvent** category
6. Select **Send to Log Analytics workspace** as the destination
7. Choose your Log Analytics workspace
8. Click **Save**

---

## Access Policy to Azure RBAC Comparison Tool

> [!IMPORTANT]
> This tool is build and maintained by Microsoft Community members and without formal Customer Support Services support.
> The tool is provided AS IS without warranty of any kind.

[PowerShell tool](https://github.com/Azure/KeyVault-AccessPolicyToRBAC-CompareTool) to compare Key Vault access policies to assigned RBAC roles to help with Access Policy to RBAC Permission Model migration. The tool intent is to provide sanity check when migrating existing Key Vault to RBAC permission model to ensure that assigned roles with underlying data actions cover existing Access Policies.

## Troubleshooting common issues

- **Role assignment delay**: Role assignments can take several minutes to propagate. Implement retry logic in your applications.
- **Lost role assignments after recovery**: Role assignments are not preserved when a vault is recovered after soft-delete. You must recreate all role assignments after recovery.
- **Access denied errors**: Check that:
  - The correct roles are assigned at the right scope
  - The service principal or managed identity has the exact permissions needed
  - Network access rules are not blocking your connection
- **Scripts fail after migration**: Update any scripts that used access policies to use role assignments instead.

## Learn more

- [Azure RBAC Overview](/azure/role-based-access-control/overview)
- [Custom Roles Tutorial](/azure/role-based-access-control/tutorial-custom-role-cli)
- [Privileged Identity Management](/azure/active-directory/privileged-identity-management/pim-configure)
