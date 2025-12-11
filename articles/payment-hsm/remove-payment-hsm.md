---
title: Remove a commissioned Azure Payment HSM
description: Remove a commissioned Azure Payment HSM
services: payment-hsm
author: msmbaldwin
ms.service: azure-payment-hsm
ms.topic: overview
ms.date: 04/10/2025
ms.author: mbaldwin

---
# Tutorial: Remove a commissioned payment HSM

Before deleting a commissioned payment HSM, decommission it first. For information on lifecycle management and deallocation scenarios, see [Azure Payment HSM lifecycle management](lifecycle-management.md).

In this tutorial, you learn how to:

> [!div class="checklist"]
> * Remove a commissioned payment HSM
> * Verify that the payment HSM is deleted

For information about creating payment HSMs, see [Tutorial: Create a payment HSM](create-payment-hsm.md).

## Remove a payment HSM from the payShield manager

Navigate to the payShield manager by following the steps in [Access the payShield manager](access-payshield-manager.md). From there, select **Remove device**.

:::image type="content" source="./media/payshield-manager-remove-device.png" alt-text="Screenshot of the payShield manager for Azure Payment HSM, remove device screen.":::

### Prepare the HSM for release

Before releasing the HSM, complete the following steps:

1. **Remove certificate data**: For HSMs running firmware versions earlier than 2.1a (1.15.0), use the `SV` command on the virtual console to view loaded certificates. Then, use the `SD` command to delete all certificate data. While residual certificates pose no security risk, remove them as a best practice. For more details, see [Known issues: TLS certificates not removed during HSM release](known-issues.md#tls-certificates-not-removed-during-hsm-release).

1. **Set HSM to Secure state**: The payment HSM must be in a Secure state before the RELEASE button is enabled. To set this state, sign in by using both Left and Right Keys and change the state to Secure.

If you need assistance with these steps, contact [Thales Support](support-guide.md). For information on firmware versions, see [Support guide: Firmware and license support](support-guide.md#firmware-and-license-support).

## Delete the payment HSM

After you release the payment HSM, delete it by using Azure CLI or Azure PowerShell.

# [Azure CLI](#tab/azure-cli)

Use the [az dedicated-hsm delete](/cli/azure/dedicated-hsm#az-dedicated-hsm-delete) command to remove your payment HSM. The following example deletes the `myPaymentHSM` payment HSM from the `myResourceGroup` resource group:

```azurecli-interactive
az dedicated-hsm delete --name "myPaymentHSM" -g "myResourceGroup"
```

You can verify that the payment HSM was deleted by using the Azure CLI [az dedicated-hsm show](/cli/azure/dedicated-hsm#az-dedicated-hsm-show) command.

```azurecli-interactive
az dedicated-hsm show --resource-group "myResourceGroup" --name "myPaymentHSM"
```

This command returns a "resource not found" error.

# [Azure PowerShell](#tab/azure-powershell)

Use the Azure PowerShell [Remove-AzDedicatedHsm](/powershell/module/az.dedicatedhsm/remove-azdedicatedhsm) cmdlet to remove your payment HSM. The following example deletes the `myPaymentHSM` payment HSM from the `myResourceGroup` resource group:

```azurepowershell-interactive
Remove-AzDedicatedHsm -Name "myPaymentHSM" -ResourceGroupName "myResourceGroup"
```

You can verify that the payment HSM was deleted by using the Azure PowerShell [Get-AzDedicatedHsm](/powershell/module/az.dedicatedhsm/get-azdedicatedhsm) cmdlet.

```azurepowershell-interactive
Get-AzDedicatedHsm -Name "myPaymentHSM" -ResourceGroup "myResourceGroup"
```

This command returns a "resource not found" error.

---

## Next steps

- Read an [Overview of Payment HSM](overview.md).
- Find out how to [get started with Azure Payment HSM](getting-started.md).
- Learn how to [Create a payment HSM](create-payment-hsm.md).
- Review [Azure Payment HSM lifecycle management](lifecycle-management.md).
- [Access the payShield manager for your payment HSM](access-payshield-manager.md).
- Understand [Azure Payment HSM support](support-guide.md).
- Review [Known issues](known-issues.md).