---
title: Virtual network service endpoints for Azure Key Vault
description: Learn how virtual network service endpoints for Azure Key Vault allow you to restrict access to a specified virtual network, including usage scenarios.
services: key-vault
author: msmbaldwin
ms.author: mbaldwin
ms.date: 05/19/2026
ms.service: azure-key-vault
ms.subservice: general
ms.topic: feature-guide
ai-usage: ai-assisted
---

# Virtual network service endpoints for Azure Key Vault

The virtual network service endpoints for Azure Key Vault allow you to restrict access to a specified virtual network. The endpoints also allow you to restrict access to a list of IPv4 (internet protocol version 4) address ranges. Any user connecting to your key vault from outside those sources is denied access.

There is one important exception to this restriction. If you opt in to allow trusted Microsoft services, Key Vault honors a trusted-mode claim that certain first-party Microsoft services—such as Microsoft 365 Exchange Online, Microsoft 365 SharePoint Online, Azure Resource Manager, and Azure Backup—include in their managed identity tokens. Services that present this claim bypass the firewall when the option is enabled. The [Trusted services](#microsoft-services-known-to-bypass-the-firewall-today) section later in this article lists the services that currently use this mechanism; Microsoft services that aren't in that table don't bypass the firewall today, even when the option is enabled. Services that do bypass the firewall still need to present a valid Microsoft Entra token, and must have permissions (configured as Azure RBAC role assignments or access policies) to perform the requested operation. For more information, see [Virtual network service endpoints](/azure/virtual-network/virtual-network-service-endpoints-overview).

## Usage scenarios

You can configure [Key Vault firewalls and virtual networks](network-security.md) to deny access to traffic from all networks (including internet traffic) by default. You can grant access to traffic from specific Azure virtual networks and public internet IP address ranges, allowing you to build a secure network boundary for your applications.

> [!NOTE]
> Key Vault firewalls and virtual network rules only apply to the [data plane](secure-key-vault.md#identity-and-access-management) of Key Vault. Key Vault control plane operations (such as create, delete, and modify operations, setting access policies, setting firewalls, and virtual network rules and deployment of secrets or keys through ARM templates) are not affected by firewalls and virtual network rules.

Here are some examples of how you might use service endpoints:

* You are using Key Vault to store encryption keys, application secrets, and certificates, and you want to block access to your key vault from the public internet.
* You want to lock down access to your key vault so that only your application, or a short list of designated hosts, can connect to your key vault.
* You have an application running in your Azure virtual network, and this virtual network is locked down for all inbound and outbound traffic. Your application still needs to connect to Key Vault to fetch secrets or certificates, or use cryptographic keys.


## Grant access to trusted Azure services 

You can grant access to trusted Azure services to the key vault, while maintaining network rules for other apps. These trusted services will then use strong authentication to securely connect to your key vault. 

You can grant access to trusted Azure services by configuring networking settings. For step-by-step guidance, see [Configure network security for Azure Key Vault](network-security.md#key-vault-firewall-enabled-trusted-services-only).

When you grant access to trusted Azure services, you grant the following types of access: 
* Trusted access for select operations to resources that are registered in your subscription. 
* Trusted access to resources based on a managed identity. 
* Trusted access across tenants using a Federated Identity Credential 

The bypass is determined by the caller's identity, not by a static list maintained inside Key Vault. When a first-party Microsoft service calls Key Vault, Key Vault inspects the caller's managed identity token. If the token carries a trusted-mode claim asserting that the caller is part of the Azure trusted subsystem, and you have enabled **Allow trusted Microsoft services to bypass this firewall**, the call is admitted. Services that don't request this claim—including Microsoft services that authenticate with a managed identity but haven't onboarded the trusted-mode claim—aren't admitted by this option and need a firewall IP rule, a virtual network rule, or a private endpoint to reach the vault.

How the bypass behaves under each `publicNetworkAccess` setting:

* `Enabled` (firewall on, selected networks): services that present the trusted-mode claim bypass the firewall.
* `Disabled` (public access disabled): services that present the trusted-mode claim still bypass and don't require a private endpoint to reach the vault. Other services are blocked.
* `SecuredByPerimeter` (vault associated with a Network Security Perimeter in Enforced mode): the bypass is overridden, and even trusted services are blocked unless an explicit perimeter access rule admits them. For details, see [Network security perimeter](network-security.md#network-security-perimeter).

<a name="trusted-services"></a>
### Microsoft services known to bypass the firewall today

The following table is an informational snapshot of Microsoft services that currently request the trusted-mode claim and therefore bypass the Key Vault firewall when the **Allow trusted services** option is enabled. The list is maintained based on team reports and might lag behind actual behavior; membership is determined by which services request the claim, not by Key Vault. If a service you're using doesn't appear here, check that service's own documentation for guidance on accessing Key Vault behind a firewall, or use a firewall rule, virtual network rule, or private endpoint.

|Trusted service|Supported usage scenarios|
| --- | --- |
| Azure API Management|[Deploy certificates for Custom Domain from Key Vault using MSI](/azure/api-management/api-management-howto-use-managed-service-identity#use-ssl-tls-certificate-from-azure-key-vault)|
| Azure App Service|App Service is trusted only for [Deploying Azure Web App Certificate through Key Vault](https://azure.github.io/AppService/2016/05/24/Deploying-Azure-Web-App-Certificate-through-Key-Vault.html), for individual app itself, the outbound IPs can be added in Key Vault's IP-based rules|
| Azure Application Gateway |[Using Key Vault certificates for HTTPS-enabled listeners](/azure/application-gateway/key-vault-certs)
| Azure Backup|Allow backup and restore of relevant keys and secrets during Azure Virtual Machines backup, by using [Azure Backup](/azure/backup/backup-overview).|
| Azure Batch | [Configure customer-managed keys for Batch accounts](/azure/batch/batch-customer-managed-key) and [Key Vault for User Subscription Batch accounts](/azure/batch/batch-account-create-portal) |
| Azure Bot Service | [Azure AI Bot Service encryption for data at rest](/azure/bot-service/bot-service-encryption#grant-azure-bot-service-access-to-a-key-vault) |
| Azure CDN | [Configure HTTPS on an Azure CDN custom domain: Grant Azure CDN access to your key vault](/azure/cdn/cdn-custom-ssl?tabs=option-2-enable-https-with-your-own-certificate#set-up-managed-identity-for-azure-cdn)|
| Azure Container Registry|[Registry encryption using customer-managed keys](/azure/container-registry/tutorial-enable-customer-managed-keys)
| Azure Data Factory|[Fetch data store credentials in Key Vault from Data Factory](https://go.microsoft.com/fwlink/?linkid=2109491)|
| Azure Data Lake Store|[Encryption of data in Azure Data Lake Store](/azure/data-lake-store/data-lake-store-encryption) with a customer-managed key.|
| Azure Data Manager for Agriculture | [Store and use your own license keys](/azure/data-manager-for-agri/concepts-byol-and-credentials) 
| Azure Database for MySQL Single server| [Data encryption for Azure Database for MySQL Single server](/azure/mysql/howto-data-encryption-cli) |
| Azure Database for MySQL Flexible server | [Data encryption for Azure Database for MySQL Flexible server](/azure/mysql/flexible-server/concepts-customer-managed-key) |
| Azure Database for PostgreSQL Single server | [Data encryption for Azure Database for PostgreSQL Single server](/azure/postgresql/howto-data-encryption-cli) |
| Azure Database for PostgreSQL Flexible server | [Data encryption for Azure Database for PostgreSQL Flexible server](/azure/postgresql/flexible-server/concepts-data-encryption) |
| Azure Databricks|[Fast, easy, and collaborative Apache Spark–based analytics service](/azure/databricks/scenarios/what-is-azure-databricks)|
| Azure Disk Encryption volume encryption service|Allow access to BitLocker Key (Windows VM) or DM Passphrase (Linux VM), and Key Encryption Key, during virtual machine deployment. This enables [Azure Disk Encryption](/azure/security/fundamentals/encryption-overview).|
| Azure Disk Storage | When configured with a Disk Encryption Set (DES). For more information, see [Server-side encryption of Azure Disk Storage using customer-managed keys](/azure/virtual-machines/disk-encryption#customer-managed-keys).|
| Azure Event Hubs|[Allow access to a key vault for customer-managed keys scenario](/azure/event-hubs/configure-customer-managed-key)|
| Azure ExpressRoute | [When using MACsec with ExpressRoute Direct](/azure/expressroute/expressroute-howto-macsec)|
| Azure Firewall Premium| [Azure Firewall Premium certificates](/azure/firewall/premium-certificates)|
| Azure Front Door Classic|[Using Key Vault certificates for HTTPS](/azure/frontdoor/front-door-custom-domain-https#prepare-your-key-vault-and-certificate)
| Azure Front Door Standard/Premium|[Using Key Vault certificates for HTTPS](/azure/frontdoor/standard-premium/how-to-configure-https-custom-domain#prepare-your-key-vault-and-certificate)
| Azure Import/Export| [Use customer-managed keys in Azure Key Vault for Import/Export service](/azure/import-export/storage-import-export-encryption-key-portal)
| Azure Information Protection|Allow access to tenant key for [Azure Information Protection.](/azure/information-protection/what-is-information-protection)|
| Azure Machine Learning|[Secure Azure Machine Learning in a virtual network](/azure/machine-learning/how-to-secure-workspace-vnet)|
| Azure Monitor | Log Analytics dedicated cluster [access to a key vault for customer-managed keys scenario](/azure/azure-monitor/logs/customer-managed-keys?tabs=portal) |
| Azure NetApp Files | [Allow access customer-managed keys in Azure Key Vault](/azure/azure-netapp-files/configure-customer-managed-keys) |
| Azure Policy Scan| Control plane policies for secrets, keys stored in data plane |
| Azure Resource Manager template deployment service|[Pass secure values during deployment](/azure/azure-resource-manager/templates/key-vault-parameter).|
| Azure Service Bus|[Allow access to a key vault for customer-managed keys scenario](/azure/service-bus-messaging/configure-customer-managed-key)|
| Azure SQL Database|[Transparent Data Encryption with Bring Your Own Key support for Azure SQL Database and Azure Synapse Analytics](/azure/azure-sql/database/transparent-data-encryption-byok-overview).|
| Azure Storage|[Storage Service Encryption using customer-managed keys in Azure Key Vault](/azure/storage/common/customer-managed-keys-configure-key-vault).|
| Azure Synapse Analytics|[Encryption of data using customer-managed keys in Azure Key Vault](/azure/synapse-analytics/security/workspaces-encryption)|
| Azure Virtual Machines deployment service|[Deploy certificates to VMs from customer-managed Key Vault](/azure/virtual-machines/extensions/key-vault-windows).|
|Exchange Online, SharePoint Online, M365DataAtRestEncryption | Allow access to customer managed keys for Data-At-Rest Encryption with [Customer Key](/microsoft-365/compliance/customer-key-overview?view=o365-worldwide&preserve-view=true).|
| Microsoft PowerPlatform | [Encryption of data in Power Platform using customer-managed keys](/power-platform/admin/customer-managed-key).
| Microsoft Purview|[Using credentials for source authentication in Microsoft Purview](/azure/purview/manage-credentials)

> [!NOTE]
> You must set up the relevant Azure RBAC for Key Vault role assignments or access policies (legacy) to allow the corresponding services to get access to Key Vault.

## Next steps

- For step-by-step instructions, see [Configure network security for Azure Key Vault](network-security.md).
- For security best practices, see [Secure your Azure Key Vault](secure-key-vault.md).
