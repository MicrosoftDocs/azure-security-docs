---
author: msmbaldwin
ms.service: azure-key-vault
ms.topic: include
ms.date: 10/30/2024
ms.author: msmbaldwin

# Used by articles that show how to assign a Key Vault access policy

---

> [!IMPORTANT]
> When using the Access Policy permission model, a user with the `Contributor`, `Key Vault Contributor`, or any other role that includes `Microsoft.KeyVault/vaults/write` permissions for the key vault management plane can grant themselves data plane access by setting a Key Vault access policy. To prevent unauthorized access and management of your key vaults, keys, secrets, and certificates, it's essential to limit Contributor role access to key vaults under the Access Policy permission model. To mitigate this risk, we recommend you use the **Role-Based Access Control (RBAC) permission model**, which restricts permission management to the 'Owner' and 'User Access Administrator' roles, allowing a clear separation between security operations and administrative duties. See the [Key Vault RBAC Guide](/azure/key-vault/general/rbac-guide) and [What is Azure RBAC?](/azure/role-based-access-control/overview) for more information.
