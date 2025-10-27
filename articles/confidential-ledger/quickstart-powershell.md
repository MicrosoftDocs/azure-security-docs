---
title: Quickstart – Microsoft Azure confidential ledger with Azure PowerShell
description: Learn to use the Microsoft Azure confidential ledger through Azure PowerShell
author: msmbaldwin
ms.author: mbaldwin
ms.date: 04/14/2025
ms.service: azure-confidential-ledger
ms.custom: devx-track-azurepowershell
ms.topic: quickstart
---

# Quickstart: Create a confidential ledger using Azure PowerShell

Azure confidential ledger is a cloud service that provides a high integrity store for sensitive data logs and records that must be kept intact. In this quickstart, you use [Azure PowerShell](/powershell/azure/) to create a confidential ledger, view and update its properties, and delete it. For more information on Azure confidential ledger and examples of what can be stored in a confidential ledger, see [About Microsoft Azure confidential ledger](overview.md).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

In this quickstart, you create a confidential ledger with [Azure PowerShell](/powershell/azure/). If you choose to install and use PowerShell locally, this tutorial requires Azure PowerShell module version 1.0.0 or later. Type `$PSVersionTable.PSVersion` to find the version. If you need to upgrade, see [Install Azure PowerShell module](/powershell/azure/install-azure-powershell). If you're running PowerShell locally, you also need to run `Login-AzAccount` to create a connection with Azure.

## Prerequisites

[!INCLUDE [Ensure subscription owner](./includes/ensure-subscription-owner.md)]

## Create a resource group

[!INCLUDE [Create resource group](../../includes/powershell-rg-create.md)]

## Get your principal ID and tenant ID

To create a confidential ledger, use your Microsoft Entra principal ID (also called your object ID). To obtain your principal ID, use the Azure PowerShell [Get-AzADUser](/powershell/module/az.resources/get-azaduser) cmdlet, with the `-SignedIn` flag:

```azurepowershell
Get-AzADUser -SignedIn
```

Your result is listed under "Id", in the format `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

## Create a confidential ledger

Use the Azure PowerShell [New-AzConfidentialLedger](/powershell/module/az.confidentialledger/new-azconfidentialledger) command to create a confidential ledger in your new resource group.

```azurepowershell
New-AzConfidentialLedger -Name "myLedger" -ResourceGroupName "myResourceGroup" -Location "EastUS" -LedgerType "Public" -AadBasedSecurityPrincipal @{ LedgerRoleName="Administrator"; PrincipalId="aaaaaaaa-bbbb-cccc-1111-222222222222"; }

```

A successful operation returns the properties of the newly created ledger. Take note of the **ledgerUri**. In the example above, this URI is "https://myledger.confidential-ledger.azure.com".

You need this URI to transact with the confidential ledger from the data plane.

## View and update your confidential ledger properties

You can view the properties associated with your newly created confidential ledger using the Azure PowerShell [Get-AzConfidentialLedger](/powershell/module/az.confidentialledger/get-azconfidentialledger) cmdlet.

```azurepowershell
Get-AzConfidentialLedger -Name "myLedger" -ResourceGroupName "myResourceGroup"
```

To update the properties of a confidential ledger, use do so, use the Azure PowerShell [Update-AzConfidentialLedger](/powershell/module/az.confidentialledger/update-azconfidentialledger) cmdlet. For instance, to update your ledger to change your role to "Reader", run:

```azurepowershell
Update-AzConfidentialLedger -Name "myLedger" -ResourceGroupName "myResourceGroup" -Location "EastUS" -LedgerType "Public" -AadBasedSecurityPrincipal @{ LedgerRoleName="Reader"; PrincipalId="aaaaaaaa-bbbb-cccc-1111-222222222222"; }
```

If you again run [Get-AzConfidentialLedger](/powershell/module/az.confidentialledger/get-azconfidentialledger), you see that the role is updated.

```json
"ledgerRoleName": "Reader",
```

## Clean up resources

[!INCLUDE [Clean up resources](../../includes/powershell-rg-delete.md)]

## Next steps

In this quickstart, you created a confidential ledger by using the Azure PowerShell. To learn more about Azure confidential ledger and how to integrate it with your applications, continue on to the articles below.

- [Overview of Microsoft Azure confidential ledger](overview.md)
