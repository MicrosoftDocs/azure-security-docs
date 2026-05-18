---
title: Azure Quickstart - Create an Azure key vault and a certificate using Bicep | Microsoft Docs
description: Quickstart showing how to create Azure key vaults, and add a self-signed certificate to the vault using Bicep.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: quickstart
ms.custom: mvc, subject-armqs, mode-arm, devx-track-bicep
ms.date: 05/14/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store certificates in Azure.
---

# Quickstart: Create an Azure key vault and a certificate using Bicep

[Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets, such as keys, passwords, and certificates. This quickstart focuses on the process of deploying a Bicep file to create a key vault and a self-signed certificate.

[!INCLUDE [About Bicep](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-bicep-introduction.md)]

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Review the Bicep file

The template used in this quickstart is from [Azure Quickstart Templates](/samples/azure/azure-quickstart-templates/key-vault-certificate-create/).

:::code language="bicep" source="~/quickstart-templates/quickstarts/microsoft.keyvault/key-vault-certificate-create/main.bicep":::

Two Azure resources are defined in the Bicep file:

* [**Microsoft.KeyVault/vaults**](/azure/templates/microsoft.keyvault/vaults): create an Azure key vault with Azure RBAC authorization enabled (`enableRbacAuthorization: true`).
* [**Microsoft.Resources/deployments**](/azure/templates/microsoft.resources/deployments): nested deployment that runs the [create-kv-certificate](https://github.com/Azure/bicep-registry-modules/tree/main/avm/ptn/deployment-script/create-kv-certificate) registry module to create a self-signed certificate in the vault. Certificates are a data-plane resource and can't be created directly with an ARM resource type.

## Deploy the Bicep file

1. Save the Bicep file as **main.bicep** to your local computer.
1. Deploy the Bicep file by using either the Azure CLI or Azure PowerShell.

    # [CLI](#tab/CLI)

    ```azurecli
    az group create --name myResourceGroup --location eastus
    az deployment group create --resource-group myResourceGroup --template-file main.bicep --parameters vaultName=<vault-name> certificateName=myCert
    ```

    # [PowerShell](#tab/PowerShell)

    ```azurepowershell
    New-AzResourceGroup -Name myResourceGroup -Location eastus
    New-AzResourceGroupDeployment -ResourceGroupName myResourceGroup -TemplateFile ./main.bicep -vaultName "<vault-name>" -certificateName "myCert"
    ```

    ---

    > [!NOTE]
    > Replace **`<vault-name>`** with the name of the key vault, which must be globally unique within the `vault.azure.net` namespace.

    When the deployment finishes, you should see a message indicating the deployment succeeded.

## Assign a Key Vault RBAC role

The key vault created by this Bicep file uses Azure RBAC for authorization. To access certificates through the data plane (for example, by using the Azure CLI or Azure PowerShell), you need to assign yourself an appropriate role.

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az role assignment create --role "Key Vault Certificates Officer" \
    --assignee-object-id $(az ad signed-in-user show --query id -o tsv) \
    --scope $(az keyvault show --name $keyVaultName --query id -o tsv)
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
$kvId = (Get-AzKeyVault -VaultName $keyVaultName).ResourceId
$userId = (Get-AzADUser -SignedIn).Id
New-AzRoleAssignment -ObjectId $userId -RoleDefinitionName "Key Vault Certificates Officer" -Scope $kvId
```

---

> [!NOTE]
> Role assignments might take a minute or two to propagate.

## Review deployed resources

You can either use the Azure portal to check the key vault and the certificate, or use the following Azure CLI or Azure PowerShell script to list the certificate created.

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az keyvault certificate list --vault-name $keyVaultName &&
echo "Press [ENTER] to continue ..."
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
Get-AzKeyVaultCertificate -vaultName $keyVaultName
Write-Host "Press [ENTER] to continue..."
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

In this quickstart, you created a key vault and a certificate using Bicep and then validated the deployment. To learn more about Key Vault and Bicep, continue on to the articles below.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- Learn more about [Bicep](/azure/azure-resource-manager/bicep/overview)
- Review the [Key Vault security overview](../general/secure-key-vault.md)
