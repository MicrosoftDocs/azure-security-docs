---
author: msmbaldwin
ms.service: azure-key-vault
ms.topic: include
ms.date: 03/30/2026
ms.author: msmbaldwin

# Used by Python quickstarts for secrets, keys, and certificates

---

# [Azure CLI](#tab/azure-cli)

1. Use the `az group create` command to create a resource group:

    ```azurecli
    az group create --name "myResourceGroup" --location "EastUS"
    ```

    You can change "EastUS" to a location nearer to you, if you prefer.

1. Use `az keyvault create` to create the key vault:

    ```azurecli
    az keyvault create --name "<vault-name>" --resource-group "myResourceGroup" --enable-rbac-authorization true --enable-purge-protection true
    ```

    Replace `<vault-name>` with a name that's unique across all of Azure. You typically use your personal or company name along with other numbers and identifiers.

# [Azure PowerShell](#tab/azure-powershell)

1. Use the `New-AzResourceGroup` command to create a resource group:

    ```azurepowershell
    New-AzResourceGroup -Name "myResourceGroup" -Location "EastUS"
    ```

    You can change "EastUS" to a location nearer to you, if you prefer.

1. Use `New-AzKeyVault` to create the key vault:

    ```azurepowershell
    New-AzKeyVault -Name "<vault-name>" -ResourceGroupName "myResourceGroup" -Location "EastUS" -EnableRbacAuthorization $true -EnablePurgeProtection
    ```

    Replace `<vault-name>` with a name that's unique across all of Azure. You typically use your personal or company name along with other numbers and identifiers.

---