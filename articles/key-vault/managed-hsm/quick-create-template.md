---
title: Azure Quickstart - Create a Managed HSM using an Azure Resource Manager template
description: Quickstart showing how to create Azure an Azure Key Vault Managed HSM using Resource Manager template
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.date: 03/13/2026
ms.topic: quickstart
ms.service: azure-key-vault
ms.subservice: managed-hsm
tags: azure-resource-manager
ms.custom: subject-armqs, devx-track-arm-template, devx-track-azurecli
#Customer intent: As a security admin who is new to Azure, I want to create a managed HSM using an Azure Resource Manager template.
---

# Quickstart: Create a Managed HSM using an ARM template

This quickstart shows how to use an Azure Resource Manager template (ARM template) to create an Azure Key Vault managed HSM. [!INCLUDE [Managed HSM description](../includes/managed-hsm/intro.md)]  

[!INCLUDE [About Azure Resource Manager](~/reusable-content/ce-skilling/azure/includes/resource-manager-quickstart-introduction.md)]

If your environment meets the prerequisites and you're familiar with using ARM templates, select the **Deploy to Azure** button. The template opens in the Azure portal.

:::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fmanaged-hsm-create%2Fazuredeploy.json":::

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [Azure CLI prepare your environment](~/reusable-content/azure-cli/azure-cli-prepare-your-environment-no-header.md)]

## Review the template

This quickstart uses a template from [Azure Quickstart Templates](https://azure.microsoft.com/resources/templates/managed-hsm-create):

:::code language="json" source="~/quickstart-templates/quickstarts/microsoft.keyvault/managed-hsm-create/azuredeploy.json":::

The template defines the following Azure resource:

* **Microsoft.KeyVault/managedHSMs**: Create an Azure Key Vault Managed HSM.

## Deploy the template

The template requires the object ID associated with your account. To find it, use the Azure CLI [az ad user show](/cli/azure/ad/user#az-ad-user-show) command, passing your email address to the `--id` parameter. You can limit the output to the object ID only by using the `--query` parameter.

```azurecli-interactive
az ad user show --id <user-email> --query "id"
```

You might also need your tenant ID. To find it, use the Azure CLI [az ad user show](/cli/azure/account#az-account-show) command. You can limit the output to the tenant ID only by using the `--query` parameter.

 ```azurecli-interactive
 az account show --query "tenantId"
 ```

Now you can deploy the ARM template:

1. Select the following image to sign in to Azure and open a template. The template creates a Managed HSM.

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.keyvault%2Fmanaged-hsm-create%2Fazuredeploy.json":::

1. Select or enter the following values.  Unless specified, use the default value to create the Managed HSM.

    - **Subscription**: Select an Azure subscription.
    - **Resource group**: Select **Create new**, enter a name for your resource group, and then select **OK**.
    - **Location**: Select a location. For example, **East US**.
    - **managedHSMName**: Enter a name for your Managed HSM.
    - **Tenant ID**: The template function automatically retrieves your tenant ID; don't change the default value.  If there's no value, enter the Tenant ID that you retrieved earlier.
    - **initialAdminObjectIds**: Enter the Object ID that you retrieved earlier.

1. Select **Purchase**. After the Managed HSM is deployed successfully, you get a notification:

[!INCLUDE [Managed HSM billing warning](../includes/managed-hsm/billing-warning.md)]

The Azure portal is used to deploy the template. In addition to the Azure portal, you can also use Azure PowerShell, Azure CLI, and REST API. To learn about other deployment methods, see [Deploy templates](/azure/azure-resource-manager/templates/deploy-powershell).

## Validate the deployment

You can verify that the managed HSM was created by using the Azure CLI [az keyvault list](/cli/azure/keyvault#az-keyvault-list) command. Format the results as a table to make the output easier to read:

```azurecli-interactive
az keyvault list -o table
```

You see the name of your newly created managed HSM.

## Clean up resources

[!INCLUDE [Delete resource group](../../../includes/cli-rg-delete.md)]

[!INCLUDE [Managed HSM cleanup warning](../includes/managed-hsm/cleanup-warning.md)]

## Next steps

In this quickstart, you created a Managed HSM. This Managed HSM isn't fully functional until you activate it. To learn how to activate your HSM, see [Activate your Managed HSM](quick-create-cli.md#activate-your-managed-hsm).

- Read an [Overview of Managed HSM](overview.md).
- Learn about [Managing keys in a Managed HSM](key-management.md).
- Review [Secure your Azure Managed HSM deployment](secure-managed-hsm.md).
