---
title: Create an Azure key vault by using ARM template
description: This article shows how to create Azure key vaults by using an Azure Resource Manager template.
services: key-vault
author: msmbaldwin
tags: azure-resource-manager
ms.custom: devx-track-arm-template
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 04/17/2025
ms.author: mbaldwin 
#Customer intent: As a security admin who's new to Azure, I want to use Key Vault to securely store keys and passwords in Azure.
---

# How to create an Azure key vault by using a Resource Manager template

[Azure Key Vault](../general/overview.md) is a cloud service that provides a secure store for secrets like keys, passwords, and certificates. This article describes the process for deploying an Azure Resource Manager template (ARM template) to create a key vault.

> [!IMPORTANT]
> Azure RBAC is the recommended authorization model for Azure Key Vault. For more information, see [Azure RBAC for Key Vault](rbac-guide.md). The legacy access policy model has known security vulnerabilities and should not be used for new deployments.

[!INCLUDE [About Azure Resource Manager](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-introduction.md)]

## Prerequisites

To complete the steps in this article:

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you start.


## Create a Key Vault Resource Manager template

The following template shows a basic way to create a key vault. Some values are specified in the template.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Specifies the name of the key vault."
      }
    },
    "skuName": {
      "type": "string",
      "defaultValue": "Standard",
      "allowedValues": [
        "Standard",
        "Premium"
      ],
      "metadata": {
        "description": "Specifies whether the key vault is a standard vault or a premium vault."
      }
    }
   },
  "resources": [
    {
      "type": "Microsoft.KeyVault/vaults",
      "apiVersion": "2023-07-01",
      "name": "[parameters('keyVaultName')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "enabledForDeployment": false,
        "enabledForDiskEncryption": false,
        "enabledForTemplateDeployment": false,
        "tenantId": "[subscription().tenantId]",
        "enableRbacAuthorization": true,
        "enableSoftDelete": true,
        "softDeleteRetentionInDays": 90,
        "enablePurgeProtection": true,
        "sku": {
          "name": "[parameters('skuName')]",
          "family": "A"
        },
        "networkAcls": {
          "defaultAction": "Deny",
          "bypass": "AzureServices"
        }
      }
    }
  ]
}

```

For more about Key Vault template settings, see [Key Vault ARM template reference](/azure/templates/microsoft.keyvault/vaults).

> [!NOTE]
> This template uses Azure RBAC for authorization, which is the recommended approach. To grant access to Key Vault data, assign Azure RBAC roles (such as Key Vault Secrets Officer or Key Vault Crypto Officer) to users, groups, or service principals. For more information, see [Azure RBAC for Key Vault](rbac-guide.md).
>
> If you need to use legacy access policies instead, see [Assign a Key Vault access policy](assign-access-policy.md). Note that the legacy access policy model has known security vulnerabilities and lacks support for Privileged Identity Management (PIM).

## More Key Vault Resource Manager templates

There are other Resource Manager templates available for Key Vault objects:

| Secrets | Keys | Certificates |
|--|--|--|
|<ul><li>[Quickstart](../secrets/quick-create-template.md)<li>[Reference](/azure/templates/microsoft.keyvault/vaults/secrets)|N/A|N/A|

You can find more Key Vault templates here: [Key Vault Resource Manager reference](/azure/templates/microsoft.keyvault/allversions).

## Deploy the templates

You can use the Azure portal to deploy the preceding templates by using the **Build your own template in editor** option as described here:
[Deploy resources from a custom template](/azure/azure-resource-manager/templates/deploy-portal#deploy-resources-from-custom-template).

You can also save the preceding templates to files and use these commands:  [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) and [az deployment group create](/cli/azure/deployment/group#az-deployment-group-create):

```azurepowershell
New-AzResourceGroupDeployment -ResourceGroupName ExampleGroup -TemplateFile key-vault-template.json
```

```azurecli
az deployment group create --resource-group ExampleGroup --template-file key-vault-template.json
```

## Clean up resources

If you plan to continue with subsequent quickstarts and tutorials, you can leave these resources in place. When you don't need the resources any longer, delete the resource group. If you delete the group, the key vault and related resources are also deleted. To delete the resource group by using the Azure CLI or Azure PowerShell, complete these steps:

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

## Resources

- Read an [Overview of Azure Key Vault](../general/overview.md).
- Learn more about [Azure Resource Manager](/azure/azure-resource-manager/management/overview).
- Review the [Azure Key Vault security overview](secure-key-vault.md)

## Next steps

- [Secure access to a key vault](secure-key-vault.md)
- [Authenticate to a key vault](authentication.md)
- [Azure Key Vault Developer's Guide](developers-guide.md)
