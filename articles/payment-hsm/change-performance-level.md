---
title: How to change the performance level of an Azure Payment HSM
description: How to change the performance level of an Azure Payment HSM
services: payment-hsm
author: msmbaldwin
ms.service: azure-payment-hsm
ms.topic: overview
ms.date: 01/30/2024
ms.author: mbaldwin

---
# How to change the performance level of a payment HSM

Azure Payment HSM supports several SKUs; for a list, see [Azure Payment HSM overview: supported SKUs](support-guide.md#supported-skus). The SKU you specify during the creation process initially determines the performance license level of your payment HSM.

You can change performance level of an existing payment HSM by changing its SKU. There is no interruption in your production payment HSMs while performance level is being updated.

The SKU of a payment HSM can be updated through ARMClient and PowerShell.

## Updating the SKU via ARMClient

You can update the SKU of your payment HSM using the [Azure Resource Manager client tool](https://github.com/projectkudu/ARMClient), which is a simple command line tool that calls the Azure Resource Manager API. Installation instructions are at <https://github.com/projectkudu/ARMClient>.

Once installed, you can use the following command:

```bash
armclient PATCH <resource-id>?api-version=2021-11-30 "{ 'sku': { 'name': '<sku>' } }" 
```

For example:

```bash
armclient PATCH /subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/myResourceGroup/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/myPaymentHSM?api-version=2021-11-30 "{ 'sku': { 'name': 'payShield10K_LMK1_CPS60' } }"
```

## Updating the SKU directly via PowerShell

You can update the SKU of your payment HSM using the Azure PowerShell [Invoke-RestMethod](/powershell/module/microsoft.powershell.utility/invoke-restmethod) cmdlet:

```azurepowershell-interactive
$sku="<sku>" 
$resourceId="<resource-id>" 
Invoke-RestMethod -Headers @{Authorization = "Bearer $((Get-AzAccessToken).Token)"} -Method PATCH -Uri "https://management.azure.com$($resourceId)?api-version=2021-11-30" -ContentType application/json -Body "{ 'sku': { 'name': '$sku' } }" 
```

For example:

```azurepowershell-interactive
$sku="payShield10K_LMK1_CPS60" 
$resourceId="/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/myResourceGroup/providers/Microsoft.HardwareSecurityModules/dedicatedHSMs/myPaymentHSM" 
Invoke-RestMethod -Headers @{Authorization = "Bearer $((Get-AzAccessToken).Token)"} -Method PATCH -Uri "https://management.azure.com$($resourceId)?api-version=2021-11-30" -ContentType application/json -Body "{ 'sku': { 'name': '$sku' } }" 
```

## Next steps

- Read an [Overview of Payment HSM](overview.md)
- Find out how to [get started with Azure Payment HSM](getting-started.md)
- See the [Azure Payment HSM frequently asked questions](faq.yml)
