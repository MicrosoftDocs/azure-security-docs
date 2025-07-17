---

title: Azure Cloud HSM Integration Guides
description: Get step-by-step integration guides for securely connecting applications and systems to Azure Cloud HSM.
services: cloud-hsm
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp

#customer intent: As an IT pro decision-maker, I want detailed integration guides so that I can connect applications and systems to Azure Cloud HSM in a way that meets security requirements.

---

# Azure Cloud HSM integration guides

Microsoft Azure Cloud HSM offers a suite of integration guides to help customers securely connect their applications and systems to Azure hardware security modules. These guides provide detailed, step-by-step instructions for integrating Azure Cloud HSM with widely used platforms and technologies, including:

- Active Directory Certificate Services (AD CS)
- Java Cryptography Extension (JCE)
- OpenSSL
- Oracle Transparent Data Encryption (TDE)
- PKCS#11
- SQL Server Extensible Key Management (EKM)
- Certificate Storage
- Offloading of Secure Sockets Layer (SSL) or Transport Layer Security (TLS) for web servers like Apache and NGINX

Each guide includes prerequisites, system requirements, configuration steps, and validation procedures tailored to a specific integration scenario. For example, the guide for SSL/TLS offloading details how to configure Apache and NGINX to use Azure Cloud HSM for cryptographic operations, to help ensure secure and efficient SSL/TLS handshakes.

To explore all available integration resources, go to the [GitHub repository](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides).

## Available integration guides

Here are the integration guides that can help you securely connect applications and systems to Azure Cloud HSM:

- [Integrating Active Directory Certificate Services with Azure Cloud HSM (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20ADCS%20Integration%20Guide.pdf)
- [Integrating JCE with Azure Cloud HSM (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20JCE%20Integration%20Guide.pdf)
- [Integrating OpenSSL with Azure Cloud HSM (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20OpenSSL%20Integration%20Guide.pdf)
- [Integrating Oracle TDE with Azure Cloud HSM (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20Oracle%20TDE%20Integration%20Guide.pdf)
- [Integrating PKCS#11 with Azure Cloud HSM (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20PKCS11%20Integration%20Guide.pdf)
- [Integrating SQL Server EKM with Azure Cloud HSM (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20SQL%20EKM%20Integration%20Guide.pdf)
- [Integrating Certificate Storage with Azure Cloud HSM](https://learn.microsoft.com/en-us/azure/cloud-hsm/integration-guides)
- [Azure Cloud HSM SSL/TLS Offloading Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20SSL%20TLS%20Offloading%20Integration%20Guide.pdf)

## Related content

- [Azure Cloud HSM overview](overview.md)
- [Deploy Azure Cloud HSM by using the Azure portal](quickstart-portal.md)
- [Deploy Azure Cloud HSM by using Azure PowerShell](quickstart-powershell.md)
- [Azure Cloud HSM onboarding guide](onboarding-guide.md)