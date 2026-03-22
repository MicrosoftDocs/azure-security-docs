---
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: include
ms.date: 03/13/2026
# Include: Managed HSM role assignment CLI example
---

## Assign Managed HSM roles

For your application to access keys, assign the appropriate Managed HSM local RBAC role to your managed identity:

```azurecli
# Get the principal ID of your managed identity
principalId=$(az vm identity show --name myVM --resource-group myRG --query principalId -o tsv)

# Assign the Crypto User role for key operations
az keyvault role assignment create \
    --hsm-name ContosoMHSM \
    --role "Managed HSM Crypto User" \
    --assignee $principalId \
    --scope /keys
```

For more information on roles and permissions, see [Managed HSM local RBAC built-in roles](/azure/key-vault/managed-hsm/built-in-roles).
