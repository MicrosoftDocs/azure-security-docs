---
title: Azure Quickstart - Set and retrieve a secret from Key Vault using Azure portal | Microsoft Docs
description: Quickstart showing how to set and retrieve a secret from Azure Key Vault using the Azure portal
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.date: 04/14/2025

ms.author: mbaldwin
ms.custom: sfi-image-nochange
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure
---
# Quickstart: Set and retrieve a secret from Azure Key Vault using the Azure portal

Azure Key Vault is a cloud service that provides a secure store for secrets. You can securely store keys, passwords, certificates, and other secrets. Azure key vaults may be created and managed through the Azure portal. In this quickstart, you create a key vault, then use it to store a secret. 

For more information, see [Key Vault Overview](../general/overview.md) and [Secrets Overview](about-secrets.md).

## Prerequisites

To access Azure Key Vault, you'll need an Azure subscription. If you don't already have a subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

All access to secrets takes place through Azure Key Vault. For this quickstart, create a key vault using the [Azure portal](../general/quick-create-portal.md), [Azure CLI](../general/quick-create-cli.md), or [Azure PowerShell](../general/quick-create-powershell.md).

## Sign in to Azure

Sign in to the [Azure portal](https://portal.azure.com).

## Add a secret to Key Vault

To add a secret to the vault, follow the steps:

1. Navigate to your key vault in the Azure portal:
1. On the Key Vault left-hand sidebar, select **Objects** then select **Secrets**.
1. Select **+ Generate/Import**.
1. On the **Create a secret** screen choose the following values:
    - **Upload options**: Manual.
    - **Name**: Type a name for the secret. The secret name must be unique within a Key Vault. The name must be a 1-127 character string, starting with a letter and containing only 0-9, a-z, A-Z, and -. For more information on naming, see [Key Vault objects, identifiers, and versioning](../general/about-keys-secrets-certificates.md#objects-identifiers-and-versioning)
    - **Value**: Type a value for the secret. Key Vault APIs accept and return secret values as strings.
    - Leave the other values to their defaults. Select **Create**.

Once you receive the message that the secret has been successfully created, you may select on it on the list.

For more information on secrets attributes, see [About Azure Key Vault secrets](./about-secrets.md)

## Retrieve a secret from Key Vault

If you select on the current version, you can see the value you specified in the previous step.

:::image type="content" source="../media/quick-create-portal/current-version-hidden.png" alt-text="Secret properties":::

By clicking "Show Secret Value" button in the right pane, you can see the hidden value.

:::image type="content" source="../media/quick-create-portal/current-version-shown.png" alt-text="Secret value appeared":::

You can also use [Azure CLI](), or [Azure PowerShell]() to retrieve previously created secret.

## Clean up resources

Other Key Vault quickstarts and tutorials build upon this quickstart. If you plan to continue on to work with subsequent quickstarts and tutorials, you may wish to leave these resources in place.
When no longer needed, delete the resource group, which deletes the Key Vault and related resources. To delete the resource group through the portal:

1. Enter the name of your resource group in the Search box at the top of the portal. When you see the resource group used in this quickstart in the search results, select it.
2. Select **Delete resource group**.
3. In the **TYPE THE RESOURCE GROUP NAME:** box type in the name of the resource group and select **Delete**.

> [!NOTE]
> It is important to notice that once a secret, key, certificate, or key vault is deleted, it will remain recoverable for a configurable period of 7 to 90 calendar days. If no configuration is specified the default recovery period will be set to 90 days. This provides users with sufficient time to notice an accidental secret deletion and respond. For more information about deleting and recovering key vaults and key vault objects, see [Azure Key Vault soft-delete overview](../general/soft-delete-overview.md)

## Next steps

In this quickstart, you created a Key Vault and stored a secret in it. To learn more about Key Vault and how to integrate it with your applications, continue on to these articles.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- Read [Secure access to a Key Vault](../general/security-features.md)
- See [Use Key Vault with App Service Web App](../general/tutorial-net-create-vault-azure-web-app.md)
- See [Use Key Vault with application deployed to VM](../general/tutorial-net-virtual-machine.md)
- See the [Azure Key Vault developer's guide](../general/developers-guide.md)
- Review the [Key Vault security overview](../general/security-features.md)
