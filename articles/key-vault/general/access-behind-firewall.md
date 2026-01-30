---
title: Access Key Vault behind a firewall - Azure Key Vault | Microsoft Docs
description: Learn about the ports, hosts, or IP addresses to open to enable a key vault client application behind a firewall to access a key vault.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 01/30/2026
ms.author: mbaldwin

---
# Access Azure Key Vault behind a firewall

To access a key vault from behind a firewall, your client application must be able to access multiple endpoints for the following functionalities:

- **Authentication**: Microsoft Entra endpoints for authenticating the security principal. For more information, see [Authentication in Azure Key Vault](authentication.md).
- **Management (control plane)**: Azure Resource Manager endpoints for creating, reading, updating, deleting, and configuring key vaults.
- **Data plane access**: Key Vault-specific endpoints (for example, `https://yourvaultname.vault.azure.net`) for accessing and managing keys, secrets, and certificates.

Depending on your configuration and environment, there are some variations.

> [!NOTE]
> For comprehensive network security guidance, including firewall configuration options from most to least restrictive, see [Secure your Azure Key Vault: Network security](secure-key-vault.md#network-security) and [Network security for Azure Key Vault](network-security.md).

## Ports

All traffic to a key vault for all three functions (authentication, management, and data plane access) goes over HTTPS: port 443. However, there's occasionally HTTP (port 80) traffic for certificate revocation list (CRL) checks. Clients that support Online Certificate Status Protocol (OCSP) shouldn't reach CRL, but might occasionally reach CRL endpoints listed in [Azure CA details](/azure/security/fundamentals/azure-ca-details#certificate-downloads-and-revocation-lists).  

## Authentication endpoints

Key vault client applications must access Microsoft Entra endpoints for authentication. The endpoint used depends on the Microsoft Entra tenant configuration, the type of principal (user principal or service principal), and the type of account (for example, a Microsoft account or a work or school account). For more information about authentication, see [Authentication in Azure Key Vault](authentication.md).  

| Principal type | Endpoint:port |
| --- | --- |
| User using Microsoft account<br> (for example, user@hotmail.com) |login.live.com:443<br><br> **Global:**<br> login.microsoftonline.com:443<br><br> **Microsoft Azure operated by 21Vianet:**<br> login.chinacloudapi.cn:443<br><br>**Azure US Government:**<br> login.microsoftonline.us:443<br> |
| User or service principal using a work or school account with Microsoft Entra ID (for example, user@contoso.com) |**Global:**<br> login.microsoftonline.com:443<br><br> **Microsoft Azure operated by 21Vianet:**<br> login.chinacloudapi.cn:443<br><br>**Azure US Government:**<br> login.microsoftonline.us:443<br> |
| User or service principal using a work or school account, plus Active Directory Federation Services (AD FS) or other federated endpoint (for example, user@contoso.com) |All endpoints for a work or school account, plus AD FS or other federated endpoints |

For more information about authentication scenarios and flows, see [Microsoft Entra authentication flow](/entra/identity-platform/authentication-vs-authorization), [Integrating Applications with Microsoft Entra ID](/entra/identity-platform/how-to-integrate), and [Active Directory Authentication Protocols](/previous-versions/azure/dn151124(v=azure.100)).  

## Control plane endpoints

For Key Vault management operations (CRUD and setting access policy), the key vault client application must access Azure Resource Manager endpoints. For more information about the control plane versus data plane access model, see [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](rbac-guide.md#key-vault-access-model-overview).  

| Type of operation | Endpoint:port |
| --- | --- |
| Key Vault control plane operations<br> via Azure Resource Manager |**Global:**<br> management.azure.com:443<br><br> **Microsoft Azure operated by 21Vianet:**<br> management.chinacloudapi.cn:443<br><br> **Azure US Government:**<br> management.usgovcloudapi.net:443<br> |
| Microsoft Graph API |**Global:**<br> graph.microsoft.com:443<br><br> **Microsoft Azure operated by 21Vianet:**<br> graph.chinacloudapi.cn:443<br><br> **Azure US Government:**<br> graph.microsoft.com:443<br> |

## Data plane endpoints

For all key vault object (keys, secrets, and certificates) management and cryptographic operations, the key vault client must access the key vault endpoint. The endpoint DNS suffix varies depending on the location of your key vault. The key vault endpoint has the format *vault-name*.*region-specific-dns-suffix*, as described in the following table.  

| Type of operation | Endpoint:port |
| --- | --- |
| Operations including cryptographic operations on keys; creating, reading, updating, and deleting keys and secrets; setting or getting tags and other attributes on key vault objects (keys or secrets) |**Global:**<br> &lt;vault-name&gt;.vault.azure.net:443<br><br> **Microsoft Azure operated by 21Vianet:**<br> &lt;vault-name&gt;.vault.azure.cn:443<br><br> **Azure US Government:**<br> &lt;vault-name&gt;.vault.usgovcloudapi.net:443<br> |

## IP address ranges

The Key Vault service uses other Azure resources like PaaS infrastructure, so it's not possible to provide a specific range of IP addresses that Key Vault service endpoints have at any particular time. If your firewall supports only IP address ranges, see the Microsoft Azure Datacenter IP Ranges documents:
- [Public](https://www.microsoft.com/en-us/download/details.aspx?id=56519)
- [US Gov](https://www.microsoft.com/en-us/download/details.aspx?id=57063)
- [China](https://www.microsoft.com/en-us/download/details.aspx?id=57062)

Authentication and Identity (Microsoft Entra ID) is a global service and can fail over to other regions or move traffic without notice. In this scenario, add all of the IP ranges listed in [Authentication and Identity IP Addresses](https://support.office.com/article/Office-365-URLs-and-IP-address-ranges-8548a211-3fe7-47cb-abb1-355ea5aa88a2#bkmk_identity_ip) to the firewall.

## Next steps

- [Network security for Azure Key Vault](network-security.md)
- [Secure your Azure Key Vault](secure-key-vault.md)
- [Configure Azure Key Vault networking settings](how-to-azure-key-vault-network-security.md)
- [Virtual network service endpoints for Azure Key Vault](overview-vnet-service-endpoints.md)
- [Integrate Key Vault with Azure Private Link](private-link-service.md)
- [Authentication in Azure Key Vault](authentication.md)
- If you have questions about Key Vault, visit the [Microsoft Q&A question page for Azure Key Vault](/answers/topics/azure-key-vault.html).
