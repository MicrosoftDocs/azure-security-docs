---
title: Network security for Azure Key Vault Managed HSM
description: Learn about network security configurations and firewall settings for Azure Managed HSM
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: conceptual
ms.date: 11/04/2025
ms.author: mbaldwin 
---

# Network security for Azure Managed HSM

This article describes different configurations for an Azure Key Vault Managed HSM firewall. For step-by-step instructions on how to configure these settings, see [How to configure Azure Managed HSM networking settings](configure-network-security.md).

## Firewall Settings

This section describes different ways to configure an Azure Key Vault Managed HSM firewall.

### Managed HSM Firewall Disabled (Public access enabled to all networks)

By default, when you create a new managed HSM, the Azure Key Vault Managed HSM firewall is disabled. All applications and Azure services can access the managed HSM and send requests to the managed HSM. This configuration doesn't grant any user permission to perform operations on your managed HSM. The managed HSM still restricts access to keys stored in the managed HSM by requiring Microsoft Entra authentication and access permissions. For more information about managed HSM authentication, see [Azure Key Vault Managed HSM access control](access-control.md).

### Managed HSM Firewall Enabled (IPv4 Addresses and Ranges - Static IPs) (PREVIEW)

If you want to authorize a particular service to access managed HSM through the Managed HSM Firewall, add its IP address to the managed HSM firewall allowlist. This configuration works best for services that use static IP addresses or well-known ranges. You can add up to 10 CIDR ranges for this configuration. 

> [!NOTE]
> This feature is in preview and requires subscription enablement. If you're interested in using this feature, create a support ticket with subscription and region information.

To allow an IP address or range of an Azure resource, such as a Web App or Logic App, perform the following steps:

1.	Sign in to the Azure portal.
1.	Select the resource (specific instance of the service).
1.	Select the **Properties** blade under **Settings**.
1.	Look for the IP address field.
1.	Copy this value or range and enter it into the managed HSM firewall allowlist.

To allow an entire Azure service through the managed HSM firewall, use the list of publicly documented data center IP addresses for Azure. Find the IP addresses associated with the service you want in the region you want and add those IP addresses to the managed HSM firewall.

> [!NOTE]
> Be aware of the following configuration limitations:
> - You can add up to 10 IPv4 rules.
> - IP network rules are only allowed for public IP addresses. IP address ranges reserved for private networks (as defined in RFC 1918) aren't allowed in IP rules. 
> - Only IPv4 addresses are supported at this time.

### Managed HSM Firewall Enabled (Trusted Services)

When you enable the Managed HSM Firewall, you can select the option to **Allow Trusted Microsoft Services to bypass this firewall**. The trusted services list doesn't cover every single Azure service. For example, Azure DevOps isn't on the trusted services list. This limitation doesn't imply that services that don't appear on the trusted services list are untrusted or insecure. The trusted services list includes services where Microsoft controls all of the code that runs on the service. Since users can write custom code in Azure services such as Azure DevOps, Microsoft doesn't provide the option to create a blanket approval for the service. Furthermore, just because a service appears on the trusted service list, it doesn't mean it's allowed for all scenarios.

To determine if a service you're trying to use is on the trusted service list, see [Virtual network service endpoints for Azure Key Vault](../general/overview-vnet-service-endpoints.md). For a how-to guide, follow the instructions for Portal, Azure CLI, and PowerShell.

### Managed HSM Firewall Enabled (Private Link)

To learn how to configure a private link connection on your managed HSM, see [Integrate Managed HSM with Azure Private Link](private-link.md).

> [!IMPORTANT]
> After firewall rules take effect, users can only perform Managed HSM data plane operations when their requests originate from allowed IPv4 address ranges. This restriction also applies to accessing Managed HSM from the Azure portal. Although users can browse to a managed HSM from the Azure portal, they might not be able to list keys if their client machine isn't in the allowed list. This restriction also affects the Managed HSM Picker used by other Azure services. Users might be able to see a list of key vaults, but not list keys, if firewall rules prevent their client machine.

### Public Access Disabled (Private Endpoint Only)

To enhance network security, you can configure your managed HSM to disable public access. This configuration denies all public access and allows only connections through private endpoints.

## Restrictions & Considerations

- Setting Public Network Access to Disable can still allow trusted services.
- Azure Key Vault Managed HSM firewall rules only apply to data plane operations. Control plane operations aren't subject to the restrictions specified in firewall rules.
- To access data by using tools such as the Azure portal, you must be on a machine within the trusted boundary that you establish when configuring network security rules.

## Next steps

- [How to configure Azure Managed HSM networking settings](configure-network-security.md)
- [Integrate Managed HSM with Azure Private Link](private-link.md)
- [Secure your Azure Managed HSM deployment](secure-managed-hsm.md)
- [Access control](access-control.md)

## References

- [Azure Resource Manager template reference for Azure Key Vault Managed HSM](/azure/templates/microsoft.keyvault/managedhsms)
- [Azure CLI commands: az keyvault network-rule](/cli/azure/keyvault/network-rule)
- Azure PowerShell cmdlets: [Add-AzKeyVaultManagedHsmNetworkRule](/powershell/module/az.keyvault/add-azkeyvaultmanagedhsmnetworkrule), [Remove-AzKeyVaultManagedHsmNetworkRule](/powershell/module/az.keyvault/remove-azkeyvaultmanagedhsmnetworkrule), [Update-AzKeyVaultManagedHsmNetworkRuleSet](/powershell/module/az.keyvault/update-azkeyvaultmanagedhsmnetworkruleset)

