---
title: Overview of Azure Cloud HSM
description: Learn how Azure Cloud HSM offers cryptographic key storage within the Azure environment as a dedicated HSM service.
author: keithp
ms.service: azure-cloud-hsm
ms.topic: overview
ms.date: 03/31/2026
ms.author: keithp

#customer intent: As an IT pro decision-maker, I'm looking for key storage capability within the Azure cloud platform that meets FIPS 140-3 Level 3 certification and that gives me exclusive access to a dedicated hardware security module.

#Supported Use Cases: Azure Cloud HSM provides a secure and customer-owned HSM for storing cryptographic keys and performing cryptographic operations. It supports various applications, including PKCS#11, offloading of SSL/TLS processing, CA private key protection, and transparent data encryption. It also supports document and code signing.

#Not Supported Use Cases: Azure Cloud HSM is IaaS only. It doesn't integrate with other Azure services, doesn't have a REST API, and doesn't support encryption at rest.
---

# What is Azure Cloud HSM?

Microsoft Azure Cloud HSM is a highly available, FIPS 140-3 Level 3 validated single-tenant service that complies with industry standards. Azure Cloud HSM grants customers complete administrative authority over their hardware security modules (HSMs). It provides a secure and customer-owned HSM cluster for storing cryptographic keys and performing cryptographic operations.

Azure Cloud HSM supports various applications, including PKCS#11, offloading of Secure Sockets Layer (SSL) or Transport Layer Security (TLS) processing, certificate authority (CA) private key protection, and transparent data encryption (TDE). It also supports document and code signing.

## Why use Azure Cloud HSM?

### Fully managed solution

Many customers require administrative control of their HSM but don't want the overhead and ancillary costs that come with cluster management for high availability, patching, and maintenance. Azure Cloud HSM customers have secure, direct, end-to-end encrypted access to HSM nodes in their HSM cluster over a private, dedicated link from their virtual network.

After a customer provisions an Azure Cloud HSM cluster, the customer maintains administrative access to their HSMs. The Azure Cloud HSM service takes care of high availability, patching, and maintenance.

### Customer-owned, highly available, single-tenant HSM as a service

Azure Cloud HSM provides high availability and redundancy by grouping multiple HSMs into an HSM cluster. The service automatically synchronizes keys and policies across each HSM node.

Each HSM cluster consists of three HSM nodes. If an HSM resource becomes unavailable, member nodes for your HSM cluster are automatically and securely migrated to healthy nodes.

The Azure Cloud HSM cluster supports load balancing of cryptographic operations. Periodic HSM backups help ensure secure and simple data recovery.

**Data residency**: Cloud HSM doesn't store or process customer data outside the region in which the customer deploys the HSM instance.

### Single-tenant HSM clusters

Each Azure Cloud HSM instance is dedicated to a single customer. Each HSM cluster uses a separate customer-specific security domain that cryptographically isolates it.

## Compliance and certification

Azure Cloud HSM meets multiple industry compliance standards and certifications to help customers satisfy regulatory requirements.

### FIPS 140-3 Level 3

Many organizations have stringent industry regulations that dictate that cryptographic keys must be stored in FIPS 140-3 Level 3 validated HSMs. Azure Cloud HSM offers HSMs that are validated to meet FIPS 140-3 Level 3 standards. For procedures to verify the authenticity of your HSM, including checking the [FIPS 140-3 Level 3 certification from NIST](https://csrc.nist.gov/projects/cryptographic-module-validation-program/certificate/4700), refer to the [onboarding guide](onboarding-guide.md). Azure Cloud HSM helps customers from various industry segments (financial services industry, government agencies, and others) meet these FIPS requirements.

### eIDAS

Azure Cloud HSM supports eIDAS compliance under the Austrian scheme by providing secure key management, cryptographic operations, and FIPS 140-3 Level 3 validated hardware to meet stringent requirements for qualified electronic signatures and seals to help ensure regulatory compliance. Learn more in the [QSCD Certificate](https://www.a-sit.at/wp-content/uploads/2024/10/VIG-19-078-QSCD-Certificate-Final_sig-HL.pdf).

### PCI and PCI 3DS

Azure Cloud HSM provides HSMs that are validated to meet PCI and PCI 3DS standards. For more details on PCI compliance certification for Azure Cloud HSM, refer to the [PCI 3DS Attestation of Compliance (AOC)](https://servicetrust.microsoft.com/DocumentPage/c88a84bd-bb48-4fe0-b93f-b4cfb7c20ba2) in Microsoft Service Trust Center.

## Azure Cloud HSM suitability

Azure Cloud HSM supports:

- PKCS#11, OpenSSL, Java Cryptography Architecture (JCA), Java Cryptography Extension (JCE), Cryptography API: Next Generation (CNG), and key storage provider (KSP).
- Active Directory Certificate Services (AD CS).
- SSL/TLS offloading (Apache or NGINX).
- TDE (Microsoft SQL Server or Oracle).
- Certificate Storage
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

## Physical security

Azure datacenters have extensive physical and procedural security controls. The HSMs in Azure Cloud HSM are hosted in a restricted access area of the datacenter, with physical access controls and video surveillance for added security.

Azure Cloud HSM incorporates both physical and logical tamper detection and response mechanisms that initiate key deletion (zeroization) of the hardware. These measures are designed to detect tampering if the physical barrier is compromised.

HSMs are safeguarded against brute-force sign-in attacks. The system locks out cryptography officers (COs) after a set number of unsuccessful access attempts. Similarly, repeated unsuccessful attempts to access an HSM with cryptography user (CU) credentials result in locking out the user. A CO must then unlock the CU. Unlocking a CO requires the `getChallenge` command, signing the challenge with the partition owner key (`PO.key`) via OpenSSL, followed by the `unlockCO` and `changePswd` commands.

## Service operations

Azure Cloud HSM doesn't have scheduled maintenance windows. However, Microsoft might need to perform maintenance for necessary upgrades or faulty hardware replacement. Customers are notified in advance if any impact is anticipated.

## Next steps

These resources are available to help you facilitate the provisioning and configuration of HSMs into your existing virtual network environment:

- [Azure Cloud HSM SDK](https://github.com/microsoft/MicrosoftAzureCloudHSM)
- [Key management in Azure](/azure/security/fundamentals/key-management)
- [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md)
- [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md)
