---
title: Azure Cloud HSM Overview (preview)
description: Learn how Azure Cloud HSM (preview) offers cryptographic key storage within the Azure environment as a dedicated HSM service.
author: keithp
ms.service: azure-cloud-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp

#Customer Intent: As an IT Pro, decision maker I am looking for key storage capability within Azure Cloud that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer owned HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offload SSL/TLS processing, certificate authority private key protection, transparent data encryption, including document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only it does not integrate with other Azure services. Cloud HSM does not have a REST API and does not support Encryption at Rest.
---

# What is Azure Cloud HSM Preview?

Microsoft Azure Cloud HSM Preview is a highly available, FIPS 140-3 Level 3 validated single-tenant HSM service that is compliant with industry standards. Azure Cloud HSM grants customers complete administrative authority over their Hardware Security Module (HSM). It provides a secure and customer owned HSM cluster for storing cryptographic keys and performing cryptographic operations. It's the ideal solution for customers who require FIPS 140-3 Level 3 validated Hardware Security Modules and supporting various applications, including PKCS#11, offload SSL/TLS processing, certificate authority private key protection, transparent data encryption, including document and code signing.

> [!NOTE]
> Microsoft Azure Cloud HSM is available through Public Preview in West Europe and UK West. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Why use Azure Cloud HSM?

### Fully Managed

Many customers require administrative control of their HSM but don't want the overhead and ancillary costs that come with cluster management for high availability, patching and maintenance. Azure Cloud HSM customers have secure, direct end to end encrypted access to their HSM instances in their HSM cluster over a private dedicated link from their VNET. After a customer provisions an Azure Cloud HSM cluster the customer maintains administrative access to their HSM while the Azure Cloud HSM service takes care of high availability, patching and maintenance through the service.

### Customer owned, highly available, single-tenant HSM as a service

Microsoft Azure Cloud HSM provides high availability and redundancy by grouping multiple HSMs into an HSM cluster and automatically synchronizing keys and policies across each HSM instance. Each HSM cluster consists of 3 HSM partitions. If an HSM resource becomes unavailable, member partitions for your HSM cluster will be automatically and securely migrated to healthy nodes. The Azure Cloud HSM cluster supports load balancing of cryptographic operations, and periodic HSM backups ensure secure and simple data recovery.

### Single-Tenant HSM Clusters

Each Azure Cloud HSM instance is dedicated to a single customer and consists of a cluster of 3 HSM partitions. Each HSM cluster uses a separate customer-specific security domain that cryptographically isolates each customer's HSM cluster.

### FIPS 140-3 Level 3 compliance

Many organizations have stringent industry regulations that dictate that cryptographic keys must be stored in [ FIPS 140-3 Level 3](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4700) validated HSMs. Azure Cloud HSM helps customers from various industry segments, such as financial services industry, government agencies, and others meet FIPS 140-2 Level-3 requirements.

## Azure Cloud HSM suitability

### Best fit

Microsoft Azure Cloud HSM is most suitable for the following types of scenarios:

- Migrating applications from on-premises to Azure Virtual Machines.
- Migrating applications from Azure Dedicated HSM or AWS Cloud HSM.
- Supporting applications that require PKCS#11.
- Running shrink-wrapped software such as Apache/Ngnix SSL Offload, MSSQL/Oracle TDE, and ADCS in Azure Virtual Machines.

> [!NOTE]
> Microsoft Azure Cloud HSM Supports:
> 
> - PKCS#11, OpenSSL, JCA/JCE, CNG/KSP
> - ADCS (Active Directory Certificate Services)
> - SSL/TLS Offloading (Apache/Nginx)
> - MSSQL/Oracle TDE (Transparent Data Encryption)
> - Document/File/Code Signing

### Not a fit

Microsoft Azure Cloud HSM does not integrate with other PaaS/SaaS Azure services. Azure Cloud HSM is IaaS only.

Microsoft Azure Cloud HSM is not a good fit for the following type of scenarios: Microsoft cloud services that require support for encryption with customer-managed keys (such as Azure Information Protection, Azure Disk Encryption, Azure Data Lake Store, Azure Storage, and Customer Key for Office 365). For those scenarios customers should use [Azure Managed HSM](../key-vault/managed-hsm/overview.md).

> [!NOTE]
>
> - Azure Cloud HSM is *Not* a Bare-Metal HSM appliance.
> - Azure Cloud HSM is *Not* a Secret Store.
> - Azure Cloud HSM is *Not* a Certificate Lifecycle Management offering.

## Next steps

Integration Guides and [Microsoft Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM) are available to help you facilitate the provisioning and configuration of HSMs into your existing virtual network environment.

* [Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM)
* [Key Management in Azure](/azure/security/fundamentals/key-management)
* [Deploy Azure Cloud HSM using the Azure portal](quickstart-portal.md)
* [Deploy Azure Cloud HSM using Azure PowerShell](quickstart-powershell.md)
