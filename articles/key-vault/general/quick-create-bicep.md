---
title: Azure Quickstart - Create an Azure key vault using Bicep | Microsoft Docs
description: Quickstart showing how to create an Azure key vault using Bicep.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: quickstart
ms.custom: mvc, subject-armqs, mode-arm, devx-track-bicep
ms.date: 05/14/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys, secrets, and certificates in Azure.
---

# Quickstart: Create an Azure key vault using Bicep

[Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets, such as keys, passwords, and certificates. This quickstart focuses on the process of deploying a Bicep file to create a key vault.

[!INCLUDE [About Bicep](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-bicep-introduction.md)]

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Review the Bicep file

The template used in this quickstart is from [Azure Quickstart Templates](https://learn.microsoft.com/samples/azure/azure-quickstart-templates/key-vault-create/).

:::code language="bicep" source="~/quickstart-templates/quickstarts/microsoft.keyvault/key-vault-create/main.bicep":::

One Azure resource is defined in the Bicep file:

* [**Microsoft.KeyVault/vaults**](/azure/templates/microsoft.keyvault/vaults): create an Azure key vault. The template enables Azure RBAC authorization (`enableRbacAuthorization: true`), soft delete, and purge protection.

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file by using either the Azure CLI or Azure PowerShell.

    # [CLI](#tab/CLI)

    ```azurecli
    az group create --name myResourceGroup --location eastus
    az deployment group create --resource-group myResourceGroup --template-file main.bicep --parameters keyVaultName=<vault-name>
    ```

    # [PowerShell](#tab/PowerShell)

    ```azurepowershell
    New-AzResourceGroup -Name myResourceGroup -Location eastus
    New-AzResourceGroupDeployment -ResourceGroupName myResourceGroup -TemplateFile ./main.bicep -keyVaultName "<vault-name>"
    ```

    ---

    > [!NOTE]
    > Replace **`<vault-name>`** with the name of the key vault, which must be globally unique within the `vault.azure.net` namespace.

    When the deployment finishes, you should see a message indicating the deployment succeeded.

## Assign a Key Vault RBAC role

The key vault created by this Bicep file uses Azure RBAC for authorization. To create or read keys, secrets, or certificates through the data plane, you need to assign yourself an appropriate role. For example, to manage secrets, assign yourself the **Key Vault Secrets Officer** role:

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az role assignment create --role "Key Vault Secrets Officer" \
    --assignee-object-id $(az ad signed-in-user show --query id -o tsv) \
    --scope $(az keyvault show --name $keyVaultName --query id -o tsv)
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
$kvId = (Get-AzKeyVault -VaultName $keyVaultName).ResourceId
$userId = (Get-AzADUser -SignedIn).Id
New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Key Vault Secrets Officer" -Scope $kvId
```

---

> [!NOTE]
> For other built-in roles, see [Azure built-in roles for Key Vault data plane operations](rbac-guide.md#azure-built-in-roles-for-key-vault-data-plane-operations). Role assignments might take a minute or two to propagate.

## Review deployed resources

You can either use the Azure portal to check the key vault, or use the following Azure CLI or Azure PowerShell script:

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az keyvault show --name $keyVaultName
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
Get-AzKeyVault -VaultName $keyVaultName
```

---

## Clean up resources

When no longer needed, use the Azure portal, Azure CLI, or Azure PowerShell to delete the resource group and its resources.

# [CLI](#tab/CLI)

```azurecli-interactive
az group delete --name myResourceGroup
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

---

[!INCLUDE [Soft-delete note](~/reusable-content/ce-skilling/azure/includes/key-vault/key-vault-cleanup-soft-delete-note.md)]

## Next steps

In this quickstart, you created a key vault using Bicep and then validated the deployment. To learn more about Key Vault and Bicep, continue on to the articles below.

- Read an [Overview of Azure Key Vault](overview.md).
- Learn more about [Bicep](/azure/azure-resource-manager/bicep/overview).
- Review the [Azure Key Vault security overview](secure-key-vault.md).
