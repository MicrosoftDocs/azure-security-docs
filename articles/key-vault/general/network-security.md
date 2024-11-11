---
title: Configure Azure Key Vault firewalls and virtual networks - Azure Key Vault 
description: Learn about key vault networking settings
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: tutorial
ms.date: 02/20/2024
ms.author: mbaldwin 
---

# Configure Azure Key Vault firewalls and virtual networks

This document will cover the different configurations for an Azure Key Vault firewall in detail. To follow the step-by-step instructions on how to configure these settings, see [Configure Azure Key Vault networking settings](how-to-azure-key-vault-network-security.md).

For more information, see [Virtual network service endpoints for Azure Key Vault](overview-vnet-service-endpoints.md).

## Firewall Settings

This section will cover the different ways that an Azure Key Vault firewall can be configured.

### Key Vault Firewall Disabled (Default)

By default, when you create a new key vault, the Azure Key Vault firewall is disabled. All applications and Azure services can access the key vault and send requests to the key vault. This configuration doesn't mean that any user will be able to perform operations on your key vault. The key vault still restricts access to secrets, keys, and certificates stored in key vault by requiring Microsoft Entra authentication and access policy permissions. To understand key vault authentication in more detail, see [Authentication in Azure Key Vault](authentication.md). For more information, see [Access Azure Key Vault behind a firewall](access-behind-firewall.md).

### Key Vault Firewall Enabled (Trusted Services Only)

When you enable the Key Vault Firewall, you'll be given an option to 'Allow Trusted Microsoft Services to bypass this firewall.' The trusted services list does not cover every single Azure service. For example, Azure DevOps isn't on the trusted services list. **This does not imply that services that do not appear on the trusted services list are not trusted or are insecure.** The trusted services list encompasses services where Microsoft controls all of the code that runs on the service. Since users can write custom code in Azure services such as Azure DevOps, Microsoft does not provide the option to create a blanket approval for the service. Furthermore, just because a service appears on the trusted service list, doesn't mean it is allowed for all scenarios.

To determine if a service you are trying to use is on the trusted service list, see [Virtual network service endpoints for Azure Key Vault](overview-vnet-service-endpoints.md#trusted-services).
For a how-to guide, follow the instructions here for [Portal, Azure CLI and PowerShell](how-to-azure-key-vault-network-security.md)

### Key Vault Firewall Enabled (IPv4 Addresses and Ranges - Static IPs)

If you would like to authorize a particular service to access key vault through the Key Vault Firewall, you can add its IP Address to the key vault firewall allowlist. This configuration is best for services that use static IP addresses or well-known ranges. There is a limit of 1000 CIDR ranges for this case.

To allow an IP Address or range of an Azure resource, such as a Web App or Logic App, perform the following steps.

1. Sign in to the Azure portal.
1. Select the resource (specific instance of the service).
1. Select the **Properties** blade under **Settings**.
1. Look for the **IP Address** field.
1. Copy this value or range and enter it into the key vault firewall allowlist.

To allow an entire Azure service, through the Key Vault firewall, use the list of publicly documented data center IP addresses for Azure [here](https://www.microsoft.com/download/details.aspx?id=56519). Find the IP addresses associated with the service you would like in the region you want and add those IP addresses to the key vault firewall.

### Key Vault Firewall Enabled (Virtual Networks - Dynamic IPs)

If you are trying to allow an Azure resource such as a virtual machine through key vault, you may not be able to use Static IP addresses, and you may not want to allow all IP addresses for Azure Virtual Machines to access your key vault.

In this case, you should create the resource within a virtual network, and then allow traffic from the specific virtual network and subnet to access your key vault. 

1. Sign in to the Azure portal.
1. Select the key vault you wish to configure.
1. Select the 'Networking' blade.
1. Select '+ Add existing virtual network'.
1. Select the virtual network and subnet you would like to allow through the key vault firewall.

### Key Vault Firewall Enabled (Private Link)

To understand how to configure a private link connection on your key vault, please see the document [here](./private-link-service.md).

> [!IMPORTANT]
> After firewall rules are in effect, users can only perform Key Vault [data plane](security-features.md#privileged-access) operations when their requests originate from allowed virtual networks or IPv4 address ranges. This also applies to accessing Key Vault from the Azure portal. Although users can browse to a key vault from the Azure portal, they might not be able to list keys, secrets, or certificates if their client machine is not in the allowed list. This also affects the Key Vault Picker used by other Azure services. Users might be able to see a list of key vaults, but not list keys, if firewall rules prevent their client machine.

> [!NOTE]
> Be aware of the following configuration limitations:
> * A maximum of 200 virtual network rules and 1000 IPv4 rules are allowed. 
> * IP network rules are only allowed for public IP addresses. IP address ranges reserved for private networks (as defined in RFC 1918) are not allowed in IP rules. Private networks include addresses that start with **10.**, **172.16-31**, and **192.168.**. 
> * Only IPv4 addresses are supported at this time.

### Public Access Disabled (Private Endpoint Only)

To enhance network security, you can configure your vault to disable public access.  This will deny all public configurations and allow only connections through private endpoints.

### Network Security Perimeter (preview)
[Network Security Perimeter](/azure/private-link/network-security-perimeter-concepts) (preview) allows organizations to define a logical network isolation boundary for PaaS resources (for example, Azure Key Vault, Azure Storage and SQL Database) that are deployed outside your organization’s virtual networks. It restricts public network access to PaaS resources outside of the perimeter, access can be exempted by using explicit access rules for public inbound and outbound.

Currently, Network Security Perimeter is in public preview for, Azure Key Vault, Azure Storage (Blobs, Files (REST), Tables, Queues), SQL DB, and Cosmos DB. See [Transition to a Network Security Perimeter](/azure/private-link/network-security-perimeter-transition).

> [!IMPORTANT]
> Private endpoint traffic is considered highly secure and therefore isn't subject to Network Security Perimeter rules. All other traffic, including trusted services, will be subject to Network Security Perimeter rules if the key vault is associated with a perimeter.

With a network security perimeter:

* All resources inside perimeter can communicate with any other resource within perimeter.
* External access is available with the following controls:
  * Public inbound access can be approved using Network and Identity attributes of the client such as source IP addresses, subscriptions.
  * Public outbound can be approved using FQDNs (Fully Qualified Domain Names) of the external destinations.
* Diagnostic Logs are enabled for PaaS resources within perimeter for Audit and Compliance.
  
#### Restrictions & Considerations

- Setting Public Network Access to Disable still allows trusted services. Switching Public Network Access to Secure by perimeter, then forbids trusted services even if configured to allow trusted services.
- Azure Key Vault firewall rules only apply to [data plane](../../azure-resource-manager/management/control-plane-and-data-plane.md#data-plane) operations. [Control plane](../../azure-resource-manager/management/control-plane-and-data-plane.md#control-plane) operations are not subject to the restrictions specified in firewall rules.
- To access data by using tools such as the Azure portal, you must be on a machine within the trusted boundary that you establish when configuring network security rules.
- Azure Key Vault has no concept of outbound rules, you can still associate a key vault to a perimeter with outbound rules but the key vault will not use them.
  
#### Associate a Network Security Perimeter with a key vault - Azure Portal

To associate a Network Security Perimeter with a key vault using the Azure portal, follow these instructions.

#### Create a network security perimeter

> [!NOTE]
> Please do not put any personal identifiable or sensitive data in the network security perimeter rules or other network security perimeter configuration.
1. Sign in to the the Azure portal
2. From **Home**, select **Create a resource**.
3. In the search box, enter **network security perimeters**. Select **network security perimeters** from the search results.
4. In the **network security perimeters** window, select **+ Create**.
5. In the **Create a network security perimeter** window, enter the following information:
    | **Setting** | **Value** |
    | --- | --- |
    | Subscription | Select the subscription you want to use for this network security perimeter. |
    | Resource group | Select resource group you want to use for this network security perimeter |
    | Name | Enter **network-security-perimeter**. |
    | Region | Select the region in which you want your network security perimeter to be created. For this quickstart, **(US) West Central US** is used. |
    | Profile name | Enter the profile you want to use for this network security perimeter |
6. Select the **Resources** tab or **Next** to proceed to the next step.
7. Select **Inbound access rules** and select **Add inbound access rule**.
8. In the **Add inbound access rule** window, enter the following information, and select **Add**:
    | **Settings** | **Value** |
    | --- | --- |
    | Rule name | Enter **inbound-rule**. |
    | Source type | Select **IP address ranges**. |
    | Allowed Sources | Enter **10.1.0.0/16** or another internal IP address range. |
9. Select **Outbound access rules** and select **Add outbound access rule**.
10. Select **Review + create** and then **Create**.
11. Select **Go to resource** to view the newly created network security perimeter.

## Associate your Key Vault with your network security perimeter

1. Sign in to the the Azure portal and navigate to the key vault resource.
2. In the service menu, under **Settings**, select **Networking**.
3. Under **Network security perimeter**, select **Associate**.

   :::image type="content" source="../media/nsp-screenshot.png" alt-text="Screenshot showing how to associate a Network Security Perimeter with a storage account in the Azure portal." lightbox="../media/nsp-screenshot.png":::

4. Search for and select a Network Security Perimeter, select a profile, and then select **Associate**.
The Network Security Perimeter is now associated with your key vault.

#### Delete a network security perimeter
When you no longer need a network security perimeter, you remove any resources associated with the network security perimeter and then remove the perimeter following these steps:
1. From your network security perimeter, select **Resources** under **Settings**.
2. Select **key-vault-YYYYDDMM** and select **Settings>Remove** from the action bar.
3. Navigate back to the **Overview** page of your network security perimeter.
4. Select **Delete** and confirm the deletion by entering **network-security-perimeter** in the text box for the name of the resource.
   
#### Associate a Network Security Perimeter with a key vault - Azure PowerShell

To associate a Network Security Perimeter with a key vault in the Azure PowerShell, follow these [instructions](/azure/private-link/create-network-security-perimeter-powershell))

#### Associate a Network Security Perimeter with a key vault - Azure CLI

To associate a Network Security Perimeter with a key vault in the Azure CLI, follow these [instructions](/azure/private-link/create-network-security-perimeter-cli)

### Network security perimeter access modes

Network security perimeter supports two different access modes for associated resources:

| **Mode** | **Description** |
|----------------|--------|
| **Learning mode**  | This is the default access mode. In *learning* mode, network security perimeter logs all traffic to the search service that would have been denied if the perimeter was in enforced mode. This allows network administrators to understand the existing access patterns of the search service before implementing enforcement of access rules. |
| **Enforced mode**  | In *Enforced* mode, network security perimeter logs and denies all traffic that isn't explicitly allowed by access rules. |

#### Network security perimeter and key vault networking settings

The `publicNetworkAccess` setting determines the key vault's association with a network security perimeter.

* In Learning mode, the `publicNetworkAccess` setting controls public access to the resource.

* In Enforced mode, the `publicNetworkAccess` setting is overridden by the network security perimeter rules. For example, if a search service with a `publicNetworkAccess` setting of `enabled` is associated with a network security perimeter in Enforced mode, access to the search service is still controlled by network security perimeter access rules.

#### Change the network security perimeter access mode

1. Navigate to your network security perimeter resource in the portal.

2. Select **Resources** in the left-hand menu.

3. Find your key vault in the table.

4. Select the three dots in the far right of the search service row. Select **Change access mode** in the popup.`

5. Select the desired access mode and select **Apply**.

#### Enable logging network access

Follow the [Collecting resource logs for Azure Network Security Perimeter Instructions](/azure/private-link/network-security-perimeter-collect-resource-logs)

#### References
* ARM Template Reference: [Azure Key Vault ARM Template Reference](/azure/templates/Microsoft.KeyVault/vaults)
* Azure CLI commands: [az keyvault network-rule](/cli/azure/keyvault/network-rule)
* Azure PowerShell cmdlets: [Get-AzKeyVault](/powershell/module/az.keyvault/get-azkeyvault), [Add-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Add-azKeyVaultNetworkRule), [Remove-AzKeyVaultNetworkRule](/powershell/module/az.KeyVault/Remove-azKeyVaultNetworkRule), [Update-AzKeyVaultNetworkRuleSet](/powershell/module/az.KeyVault/Update-azKeyVaultNetworkRuleSet)

#### Next steps
* [Virtual network service endpoints for Key Vault](overview-vnet-service-endpoints.md)
* [Azure Key Vault security overview](security-features.md)