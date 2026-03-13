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

For more information on roles and permissions, see [Managed HSM local RBAC built-in roles](../../managed-hsm/built-in-roles.md).
