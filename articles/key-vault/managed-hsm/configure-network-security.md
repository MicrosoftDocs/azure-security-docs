---
title: Configure Azure Managed HSM networking settings using IP Network Firewall (preview)
description: Step-by-step instructions to configure Managed HSM firewalls and network access
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: how-to
ms.date: 01/30/2026
ms.author: mbaldwin 
---

# Configure Azure Managed HSM networking settings with IP Network Firewall (preview)

This article provides step-by-step guidance on how to configure Azure Key Vault Managed HSM networking settings with IP Network Firewall (preview) to work with other applications and Azure services. For detailed information about different network security configurations and concepts, see [Network security for Azure Key Vault Managed HSM](network-security.md).

Here are step-by-step instructions to configure Managed HSM firewall using the Azure portal, Azure CLI, and Azure PowerShell.

> [!NOTE]
> The IP Network Firewall (preview) feature is available in public preview in East US, West US, West US 2, West Central US, UK South, and West Europe.  

> [!IMPORTANT]
> There is a limit of 10 IP address or address ranges you can add per resource.

# [Azure portal](#tab/azure-portal)

Here's how to configure Managed HSM firewalls by using the Azure portal:

1.	Browse to the Managed HSM you want to secure.
1.	Select **Networking**, then select the **Public access** tab.
1.	Under **Public network access**, select **Manage**.
1.	To add IP addresses to firewalls, next to **Public network access**, select **Enable** and next to **Default action**, select **Enable from selected networks**.
1.	Under **IP Networks**, add IPv4 address ranges by typing IPv4 address ranges in CIDR (Classless Inter-domain Routing) notation or individual IP addresses.
1.	If you want to allow Microsoft Trusted Services to bypass the Managed HSM Firewall, select **Yes**. For a full list of the current Managed HSM Trusted Services, see [Azure Key Vault Trusted Services](/azure/key-vault/general/overview-vnet-service-endpoints#trusted-services).
1.	Select **Save**.

# [Azure CLI](#tab/azure-cli)

Here's how to configure Managed HSM firewalls by using the Azure CLI:

1. Install Azure CLI and sign in.
1. Use the [az keyvault update-hsm](/cli/azure/keyvault#az-keyvault-update-hsm) command to set the default action to Deny before creating a firewall.

   ```azurecli
   az keyvault update-hsm --resource-group "myresourcegroup" --hsm-name "mymanagedhsm" --default-action Deny
   ```

1. Use the [az keyvault network-rule add](/cli/azure/keyvault/network-rule#az-keyvault-network-rule-add) command to add an IP address range to allow traffic.

   ```azurecli
   az keyvault network-rule add --resource-group "myresourcegroup" --hsm-name "mymanagedhsm" --ip-address "191.10.18.0/24"
   ```

1. If this key vault should be accessible by any trusted services, use the [az keyvault update](/cli/azure/keyvault#az-keyvault-update) command to set bypass to AzureServices.

   ```azurecli
   az keyvault update --resource-group "myresourcegroup" --hsm-name "mymanagedhsm" --bypass AzureServices
   ```

# [Azure PowerShell](#tab/azure-powershell)

Here's how to configure Managed HSM firewalls by using PowerShell:

1. Install the latest Azure PowerShell, and sign in.
1. Use the [Update-AzKeyVaultManagedHsmNetworkRuleSet](/powershell/module/az.keyvault/update-azkeyvaultmanagedhsmnetworkruleset) cmdlet to set default action to Deny and add an IP address range to allow traffic.

   ```powershell
   Update-AzKeyVaultManagedHsmNetworkRuleSet -Name "mymanagedhsm" -ResourceGroupName "myresourcegroup" -DefaultAction Deny -IpAddressRange @('16.17.18.0/24') -PassThru 
   ```

   Include `-ReplaceAllRules` to overwrite IP Lists. Otherwise, the command merges the newly included rules.

1. If this managed HSM should be accessible by any trusted services, use the [Update-AzKeyVaultManagedHsmNetworkRuleSet](/powershell/module/az.keyvault/update-azkeyvaultmanagedhsmnetworkruleset) cmdlet to set bypass to AzureServices.

   ```powershell
   Update-AzKeyVaultManagedHsmNetworkRuleSet -Name "mymanagedhsm" -Bypass AzureServices
   ```

---

## Next steps

- [Network security for Azure Key Vault Managed HSM](network-security.md)
- [Integrate with Azure Private Link](private-link.md)
- [Secure your Azure Managed HSM deployment](secure-managed-hsm.md)
- [Access control](access-control.md)
- [Azure Resource Manager template reference for Azure Key Vault Managed HSM](/azure/templates/microsoft.keyvault/managedhsms)
