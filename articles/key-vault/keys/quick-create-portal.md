---
title: Azure Quickstart - Set and retrieve a key from Key Vault using Azure portal | Microsoft Docs
description: Quickstart showing how to set and retrieve a key from Azure Key Vault using the Azure portal
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: keys
ms.topic: quickstart
ms.custom: mvc, mode-ui
ms.date: 04/14/2025

ms.author: mbaldwin
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys in Azure
---
# Quickstart: Set and retrieve a key from Azure Key Vault using the Azure portal

Azure Key Vault is a cloud service that provides a secure store for secrets. You can securely store keys, passwords, certificates, and other secrets. Azure key vaults may be created and managed through the Azure portal. In this quickstart, you create a key vault, then use it to store a key. For more information on Key Vault, review the [Overview](../general/overview.md).

## Prerequisites

To access Azure Key Vault, you'll need an Azure subscription. If you don't already have a subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

All access to secrets takes place through Azure Key Vault. For this quickstart, create a key vault using the [Azure portal](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md), or [Azure PowerShell](../general/quick-create-powershell.md).

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Add a key to Key Vault

To add a key to the vault, you just need to take a couple of additional steps. In this case, we add a key that could be used by an application. The key is called **ExampleKey**.

1. On the Key Vault properties pages, select **Keys**.
2. Select **Generate/Import**.
3. On the **Create a key** screen choose the following values:
    - **Options**: Generate.
    - **Name**: ExampleKey.
    - Leave the other values to their defaults. Select **Create**.

## Retrieve a key from Key Vault

Once that you receive the message that the key has been successfully created, you may click on it on the list. You can then see some of the properties and select **Download public key** to retrieve the key.

:::image type="content" source="../media/keys/quick-create-portal/current-version-hidden.png" alt-text="Key properties":::

## Clean up resources

Other Key Vault quickstarts and tutorials build upon this quickstart. If you plan to continue on to work with subsequent quickstarts and tutorials, you may wish to leave these resources in place.
When no longer needed, delete the resource group, which deletes the Key Vault and related resources. To delete the resource group through the portal:

1. Enter the name of your resource group in the Search box at the top of the portal. When you see the resource group used in this quickstart in the search results, select it.
2. Select **Delete resource group**.
3. In the **TYPE THE RESOURCE GROUP NAME:** box type in the name of the resource group and select **Delete**.


## Next steps

In this quickstart, you created a Key Vault and stored a key in it. To learn more about Key Vault and how to integrate it with your applications, continue on to these articles.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- See the [Azure Key Vault developer's guide](../general/developers-guide.md)
- Review the [Key Vault security overview](../general/security-features.md)
