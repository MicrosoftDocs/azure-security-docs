---
title: Azure Quickstart - Create an Azure key vault and a secret by using Azure Resource Manager template | Microsoft Docs
description: Quickstart showing how to create Azure key vaults, and add secrets to the vaults by using Azure Resource Manager template.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.service: azure-key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, subject-armqs, mode-arm, devx-track-arm-template, sfi-image-nochange
ms.date: 04/10/2026
ms.author: mbaldwin
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure.
---

# Quickstart: Set and retrieve a secret from Azure Key Vault using an ARM template

[Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets, such as keys, passwords, certificates, and other secrets. This quickstart focuses on the process of deploying an Azure Resource Manager template (ARM template) to create a key vault and a secret.

[!INCLUDE [About Azure Resource Manager](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-introduction.md)]

If your environment meets the prerequisites and you're familiar with using ARM templates, select the **Deploy to Azure** button. The template will open in the Azure portal.

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fkey-vault-create%2Fazuredeploy.json":::

## Prerequisites

To complete this article:

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Review the template

The template used in this quickstart is from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/key-vault-create/).

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.keyvault/key-vault-create/azuredeploy.json":::

Two Azure resources are defined in the template:

* [**Microsoft.KeyVault/vaults**](/azure/templates/microsoft.keyvault/vaults): create an Azure key vault.
* [**Microsoft.KeyVault/vaults/secrets**](/azure/templates/microsoft.keyvault/vaults/secrets): create a key vault secret.

The template creates the key vault with Azure RBAC authorization enabled. This means the vault uses Azure role-based access control (Azure RBAC) for data plane authorization, rather than access policies.

More Azure Key Vault template samples can be found in [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Keyvault&pageNumber=1&sort=Popular).

## Deploy the template

1. Select the following image to sign in to Azure and open a template. The template creates a key vault and a secret.

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fkey-vault-create%2Fazuredeploy.json":::

2. Select or enter the following values.

    Unless it's specified, use the default value to create the key vault and a secret.

    * **Subscription**: select an Azure subscription.
    * **Resource group**: select **Create new**, enter a unique name for the resource group, and then click **OK**.
    * **Location**: select a location. For example, **Central US**.
    * **Key Vault Name**: enter a name for the key vault, which must be globally unique within the .vault.azure.net namespace. You need the name in the next section when you validate the deployment.
    * **Secret Name**: enter a name for the secret that you store in the key vault. For example, **adminpassword**.
    * **Secret Value**: enter the secret value. If you store a password, it's recommended to use a generated password.
    * **I agree to the terms and conditions state above**: Select.
3. Select **Purchase**. After the key vault has been deployed successfully, you get a notification:

    ![ARM template, Key Vault integration, deploy portal notification](../media/quick-create-template/resource-manager-template-portal-deployment-notification.png)

The Azure portal is used to deploy the template. In addition to the Azure portal, you can also use the Azure PowerShell, Azure CLI, and REST API. To learn other deployment methods, see [Deploy templates](/azure/azure-resource-manager/templates/deploy-powershell).

## Assign a Key Vault RBAC role

The key vault created by this template uses Azure RBAC for authorization. To access secrets through the data plane (for example, using the Azure CLI or Azure PowerShell), you need to assign yourself an appropriate role.

1. Get your Microsoft Entra user object ID:

    # [CLI](#tab/CLI)
    ```azurecli-interactive
    az ad signed-in-user show --query id -o tsv
    ```

    # [PowerShell](#tab/PowerShell)
    ```azurepowershell-interactive
    (Get-AzADUser -SignedIn).Id
    ```

    ---

2. Assign the **Key Vault Secrets Officer** role to yourself on the key vault:

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
    > Role assignments might take a minute or two to propagate.

## Review deployed resources

You can either use the Azure portal to check the key vault and the secret, or use the following Azure CLI or Azure PowerShell script to list the secret created.

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter your key vault name:" &&
read keyVaultName &&
az keyvault secret list --vault-name $keyVaultName &&
echo "Press [ENTER] to continue ..."
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$keyVaultName = Read-Host -Prompt "Enter your key vault name"
Get-AzKeyVaultSecret -vaultName $keyVaultName
Write-Host "Press [ENTER] to continue..."
```

---

## Clean up resources

Other Key Vault quickstarts and tutorials build upon this quickstart. If you plan to continue on to work with subsequent quickstarts and tutorials, you may wish to leave these resources in place.
When no longer needed, delete the resource group, which deletes the Key Vault and related resources. To delete the resource group by using Azure CLI or Azure PowerShell:

# [CLI](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

# [PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

---

## Next steps

In this quickstart, you created a key vault and a secret using an ARM template, and validated the deployment. To learn more about Key Vault and Azure Resource Manager, continue on to the articles below.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- Learn more about [Azure Resource Manager](/azure/azure-resource-manager/management/overview)
- Review the [Key Vault security overview](../general/secure-key-vault.md)
