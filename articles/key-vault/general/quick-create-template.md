---
title: Azure Quickstart - Create an Azure key vault by using ARM template | Microsoft Docs
description: Quickstart showing how to create an Azure key vault by using an Azure Resource Manager template (ARM template).
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: azure-key-vault
ms.subservice: general
ms.topic: quickstart
ms.custom: mvc, subject-armqs, mode-arm, devx-track-arm-template
ms.date: 05/14/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys, secrets, and certificates in Azure.
---

# Quickstart: Create an Azure key vault by using ARM template

[Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets, such as keys, passwords, and certificates. This quickstart focuses on the process of deploying an Azure Resource Manager template (ARM template) to create a key vault.

[!INCLUDE [About Azure Resource Manager](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-introduction.md)]

If your environment meets the prerequisites and you're familiar with using ARM templates, select the **Deploy to Azure** button. The template will open in the Azure portal.

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fkey-vault-create%2Fazuredeploy.json":::

## Prerequisites

To complete this article:

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Review the template

The template used in this quickstart is from [Azure Quickstart Templates](/samples/azure/azure-quickstart-templates/key-vault-create/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.keyvault/key-vault-create/azuredeploy.json":::

One Azure resource is defined in the template:

* [**Microsoft.KeyVault/vaults**](/azure/templates/microsoft.keyvault/vaults): create an Azure key vault. The template enables Azure RBAC authorization (`enableRbacAuthorization: true`), soft delete, and purge protection.

More Azure Key Vault template samples can be found in [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault&pageNumber=1&sort=Popular).

## Deploy the template

1. Select the following image to sign in to Azure and open a template. The template creates a key vault.

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fkey-vault-create%2Fazuredeploy.json":::

2. Select or enter the following values. Unless it's specified, use the default value.

    * **Subscription**: select an Azure subscription.
    * **Resource group**: select **Create new**, enter a unique name for the resource group, and then select **OK**.
    * **Region**: select a location. For example, **Central US**.
    * **Vault Name**: enter a name for the key vault, which must be globally unique within the `vault.azure.net` namespace.
    * **Sku Name**: select **standard** or **premium**. The default is **standard**.

3. Select **Review + create**, then select **Create**. After the key vault has been deployed successfully, you get a notification.

You can also use Azure PowerShell, the Azure CLI, or the REST API to deploy the template. To learn other deployment methods, see [Deploy templates](/azure/azure-resource-manager/templates/deploy-powershell).

## Assign a Key Vault RBAC role

The key vault created by this template uses Azure RBAC for authorization. To create or read keys, secrets, or certificates through the data plane, you need to assign yourself an appropriate role. For example, to manage secrets, assign yourself the **Key Vault Secrets Officer** role:

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

Other Key Vault quickstarts and tutorials build upon this quickstart. If you plan to continue on to work with subsequent quickstarts and tutorials, you may wish to leave these resources in place. When no longer needed, delete the resource group, which deletes the key vault and related resources.

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
```

---

[!INCLUDE [Soft-delete note](~/reusable-content/ce-skilling/azure/includes/key-vault/key-vault-cleanup-soft-delete-note.md)]

## More Key Vault Resource Manager templates

Other quickstarts walk through populating a key vault with secrets, keys, or certificates:

| Secrets | Keys | Certificates |
|--|--|--|
|<ul><li>[ARM template quickstart](../secrets/quick-create-template.md)</li><li>[Bicep quickstart](../secrets/quick-create-bicep.md)</li><li>[Reference](/azure/templates/microsoft.keyvault/vaults/secrets)</li></ul>|<ul><li>[ARM template quickstart](../keys/quick-create-template.md)</li><li>[Bicep quickstart](../keys/quick-create-bicep.md)</li><li>[Reference](/azure/templates/microsoft.keyvault/vaults/keys)</li></ul>|<ul><li>[ARM template quickstart](../certificates/quick-create-template.md)</li><li>[Bicep quickstart](../certificates/quick-create-bicep.md)</li></ul>|

You can find more Key Vault templates here: [Key Vault Resource Manager reference](/azure/templates/microsoft.keyvault/allversions).

## Next steps

- Read an [Overview of Azure Key Vault](overview.md).
- Review the [Azure Key Vault security overview](secure-key-vault.md).
- Learn about [Azure RBAC for Key Vault](rbac-guide.md).
- [Authenticate to a key vault](authentication.md).
- [Azure Key Vault Developer's Guide](developers-guide.md).
