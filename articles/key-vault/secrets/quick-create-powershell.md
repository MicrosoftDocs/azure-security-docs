---
title: Quickstart - Set & retrieve a secret from Key Vault using PowerShell
description: In this quickstart, learn how to create, retrieve, and delete secrets from an Azure Key Vault using Azure PowerShell.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: secrets
ms.topic: quickstart
ms.custom: mvc, devx-track-azurepowershell, mode-api
ms.date: 08/27/2024
ms.author: mbaldwin
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure
---

# Quickstart: Set and retrieve a secret from Azure Key Vault using PowerShell

Azure Key Vault is a cloud service that works as a secure secrets store. You can securely store keys, passwords, certificates, and other secrets. For more information on Key Vault, you may review the [Overview](../general/overview.md). In this quickstart, you use Azure PowerShell to create a key vault. You then store a secret in the newly created vault.

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

If you choose to install and use PowerShell locally, this tutorial requires Azure PowerShell module version 5.0.0 or later. Type `Get-InstalledModule -Name Az` to find the version. If you need to upgrade, see [How to install Azure PowerShell](/powershell/azure/install-azure-powershell). If you are running PowerShell locally, you also need to run `Connect-AzAccount` to create a connection with Azure.

```azurepowershell
Connect-AzAccount
```

## Create a resource group

[!INCLUDE [Create a resource group](../../../includes/powershell-rg-create.md)]

## Create a key vault

[!INCLUDE [Create a key vault](../includes/key-vault-creation-powershell.md)]

## Give your user account permissions to manage secrets in Key Vault

[!INCLUDE [Using RBAC to provide access to a key vault](../includes/rbac/upn-secrets-officer-powershell.md)]

## Adding a secret to Key Vault

To add a secret to the vault, you just need to take a couple of steps. In this case, you add a password that could be used by an application. The password is called **ExamplePassword** and stores the value of **hVFkk965BuUv** in it.

First, run the following command and enter the value **hVFkk965BuUv** when prompted to convert it to a secure string:

```azurepowershell-interactive
$secretvalue = Read-Host -Prompt 'Enter the example password' -AsSecureString
```

Then, use the Azure PowerShell [Set-AzKeyVaultSecret](/powershell/module/az.keyvault/set-azkeyvaultsecret) cmdlet to create a secret in Key Vault called **ExamplePassword** with the value **hVFkk965BuUv** :


```azurepowershell-interactive
$secret = Set-AzKeyVaultSecret -VaultName "<your-unique-keyvault-name>" -Name "ExamplePassword" -SecretValue $secretvalue
```

## Retrieve a secret from Key Vault

To view the value contained in the secret as plain text, use the Azure PowerShell [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret) cmdlet:

```azurepowershell-interactive
$secret = Get-AzKeyVaultSecret -VaultName "<your-unique-keyvault-name>" -Name "ExamplePassword" -AsPlainText
```

Now, you have created a Key Vault, stored a secret, and retrieved it.

## Clean up resources

 Other quickstarts and tutorials in this collection build upon this quickstart. If you plan to continue on to work with other quickstarts and tutorials, you may want to leave these resources in place.

When no longer needed, you can use the [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) command to remove the resource group, Key Vault, and all related resources.

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroup
```

## Next steps

In this quickstart, you created a Key Vault and stored a secret in it. To learn more about Key Vault and how to integrate it with your applications, continue on to the articles below.

- Read an [Overview of Azure Key Vault](../general/overview.md)
- Learn how to [store multiline secrets in Key Vault](multiline-secrets.md)
- See the reference for the [Azure PowerShell Key Vault cmdlets](/powershell/module/az.keyvault/#key_vault)
- Review the [Key Vault security overview](../general/security-features.md)
