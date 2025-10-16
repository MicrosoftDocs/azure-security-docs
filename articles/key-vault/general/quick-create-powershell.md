---
title: Quickstart - Create an Azure Key Vault with Azure PowerShell
description: Quickstart showing how to create an Azure Key Vault using Azure PowerShell
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: quickstart
ms.date: 08/25/2024
ms.author: mbaldwin
ms.custom: mode-api, devx-track-azurepowershell
#Customer intent: As a security admin who is new to Azure, I want to use Key Vault to securely store keys and passwords in Azure
---
# Quickstart: Create a key vault using PowerShell

In this quickstart, you create a key vault with [Azure PowerShell](/powershell/azure/). Azure Key Vault is a cloud service that provides a secure store for [keys](../keys/index.yml), [secrets](../secrets/index.yml), and [certificates](../certificates/index.yml). For more information on Key Vault, see [About Azure Key Vault](overview.md); for more information on what can be stored in a key vault, see [About keys, secrets, and certificates](about-keys-secrets-certificates.md).

## Prerequisites

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [azure-powershell-requirements-no-header.md](~/reusable-content/azure-powershell/azure-powershell-requirements-no-header.md)]

## Create a resource group

[!INCLUDE [Create a resource group](~/reusable-content/ce-skilling/azure/includes/create-resource-group-powershell.md)]

## Create a key vault

[!INCLUDE [Create a key vault](~/reusable-content/ce-skilling/azure/includes/key-vault/create-key-vault-powershell.md)]

## Clean up resources

[!INCLUDE [Clean up resources](~/reusable-content/ce-skilling/azure/includes/delete-resource-group-powershell.md)]

## Next steps

In this quickstart you created a Key Vault using Azure PowerShell. To learn more about Key Vault and how to integrate it with your applications, continue on to the articles below.

- Read an [Overview of Azure Key Vault](overview.md)
- See the reference for the [Azure PowerShell Key Vault cmdlets](/powershell/module/az.keyvault/)
- Review the [Azure Key Vault security overview](security-features.md)
