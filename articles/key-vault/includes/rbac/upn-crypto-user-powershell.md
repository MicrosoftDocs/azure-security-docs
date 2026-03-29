---
author: msmbaldwin
ms.service: azure-key-vault
ms.topic: include
ms.date: 04/14/2025

ms.author: msmbaldwin

# Used by articles that show how to assign a Key Vault access policy

---

To gain permissions to your key vault through [Role-Based Access Control (RBAC)](/azure/key-vault/general/rbac-guide.md), assign a role to your "User Principal Name" (UPN) using the Azure PowerShell cmdlet [New-AzRoleAssignment](/powershell/module/az.resources/new-azroleassignment).

```azurepowershell
New-AzRoleAssignment -SignInName "<upn>" -RoleDefinitionName "Key Vault Crypto User" -Scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.KeyVault/vaults/<vault-name>"
```

Replace `<upn>`, `<subscription-id>`, `<resource-group>`, and `<vault-name>` with your actual values. Your UPN is typically in the format of an email address (for example, username@domain.com).
