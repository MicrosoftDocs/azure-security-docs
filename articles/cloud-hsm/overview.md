---
title: Overview of Azure Cloud HSM Preview
description: Learn how Azure Cloud HSM Preview offers cryptographic key storage within the Azure environment as a dedicated HSM service.
author: keithp
ms.service: azure-cloud-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp

#customer intent: As an IT pro decision-maker, I'm looking for key storage capability within the Azure cloud platform that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer-owned HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offloading of SSL/TLS processing, CA private key protection, and transparent data encryption. It also supports document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only. It doesn't integrate with other Azure services, doesn't have a REST API, and doesn't support encryption at rest.
---

# What is Azure Cloud HSM Preview?

Microsoft Azure Cloud HSM Preview is a highly available, FIPS 140-3 Level 3 validated single-tenant service that complies with industry standards. Azure Cloud HSM grants customers complete administrative authority over their hardware security modules (HSMs). It provides a secure and customer-owned HSM cluster for storing cryptographic keys and performing cryptographic operations.

Azure Cloud HSM supports various applications, including PKCS#11, offloading of Secure Sockets Layer (SSL) or Transport Layer Security (TLS) processing, certificate authority (CA) private key protection, and transparent data encryption (TDE). It also supports document and code signing.

> [!NOTE]
> Azure Cloud HSM is available in preview in the East US, East US 2, West US, West US 2, West Europe, North Europe, and UK West regions. For legal terms that apply to Azure features that are in beta, in preview, or otherwise not yet released into general availability, see the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## Why use Azure Cloud HSM?

### Fully managed solution

Many customers require administrative control of their HSM but don't want the overhead and ancillary costs that come with cluster management for high availability, patching, and maintenance. Azure Cloud HSM customers have secure, direct, end-to-end encrypted access to their HSM instances in their HSM cluster over a private, dedicated link from their virtual network.

After a customer provisions an Azure Cloud HSM cluster, the customer maintains administrative access to their HSMs. The Azure Cloud HSM service takes care of high availability, patching, and maintenance.

### Customer-owned, highly available, single-tenant HSM as a service

Azure Cloud HSM provides high availability and redundancy by grouping multiple HSMs into an HSM cluster. The service automatically synchronizes keys and policies across each HSM instance.

Each HSM cluster consists of three HSM partitions. If an HSM resource becomes unavailable, member partitions for your HSM cluster are automatically and securely migrated to healthy nodes.

The Azure Cloud HSM cluster supports load balancing of cryptographic operations. Periodic HSM backups help ensure secure and simple data recovery.

### Single-tenant HSM clusters

Each Azure Cloud HSM instance is dedicated to a single customer. Each HSM cluster uses a separate customer-specific security domain that cryptographically isolates it.

### FIPS 140-3 Level 3 compliance

Many organizations have stringent industry regulations that dictate that cryptographic keys must be stored in [FIPS 140-3 Level 3](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4700) validated HSMs. Azure Cloud HSM helps customers from various industry segments (financial services industry, government agencies, and others) meet these FIPS requirements.

## Azure Cloud HSM suitability

Azure Cloud HSM supports:

- PKCS#11, OpenSSL, Java Cryptography Architecture (JCA), Java Cryptography Extension (JCE), Cryptography API: Next Generation (CNG), and key storage provider (KSP).
- Active Directory Certificate Services (AD CS).
- SSL/TLS offloading (Apache or NGINX).
- TDE (Microsoft SQL Server or Oracle).
- Document, file, and code signing.

Azure Cloud HSM is *not*:

- A bare-metal HSM appliance.
- A secret store.
- An offering for certificate life-cycle management.

### Best fit

Azure Cloud HSM is most suitable for the following types of scenarios:

- Migrating applications from on-premises to Azure Virtual Machines
- Migrating applications from Azure Dedicated HSM or AWS Cloud HSM
- Supporting applications that require PKCS#11
- Running shrink-wrapped software such as Apache or NGINX SSL offloading, SQL Server or Oracle TDE, and AD CS in Azure Virtual Machines

### Not a fit

Azure Cloud HSM doesn't integrate with other platform as a service (PaaS) or software as a service (SaaS) Azure services. Azure Cloud HSM is infrastructure as a service (IaaS) only.

Azure Cloud HSM is not a good fit for Microsoft cloud services that require support for encryption with customer-managed keys. These services include Azure Information Protection, Azure Disk Encryption, Azure Data Lake Storage, Azure Storage, and Microsoft Purview Customer Key. For those scenarios, customers should use [Azure Key Vault Managed HSM](../key-vault/managed-hsm/overview.md).

## Related content

These resources are available to help you facilitate the provisioning and configuration of HSMs into your existing virtual network environment:

- [Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM)
- [Key management in Azure](/azure/security/fundamentals/key-management)
- [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md)
- [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md)
