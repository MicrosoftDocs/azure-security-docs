---

title: Azure Cloud HSM Integration Guides
description: Step-by-step integration guides for securely connecting applications and systems to Azure Cloud HSM.
services: cloud-hsm
author: keithp
manager: keithp
ms.service: azure-cloud-hsm
ms.topic: overview
ms.date: 03/20/2025
ms.author: keithp

#CustomerIntent: Provide detailed integration guides for securely connecting applications and systems to Azure Cloud HSM.

---

# Azure Cloud HSM integration guides

Microsoft Azure Cloud HSM offers a suite of integration guides to help customers securely connect their applications and systems to Azure's hardware security modules. These guides provide detailed, step-by-step instructions for integrating Azure Cloud HSM with widely used platforms and technologies, including Active Directory Certificate Services (ADCS), Java Cryptography Extension (JCE), OpenSSL, Oracle Transparent Data Encryption (TDE), PKCS#11, SQL Server Extensible Key Management (EKM), and SSL/TLS Offloading for web servers like Apache and NGINX.

Each guide includes prerequisites, system requirements, configuration steps, and validation procedures tailored to the specific integration scenario. For example, the SSL/TLS Offloading guide details how to configure Apache and NGINX to leverage Azure Cloud HSM for cryptographic operations, ensuring secure and efficient SSL/TLS handshakes. To explore all available integration resources, visit the repository: [Azure Cloud HSM Integration Guides](https://github.com/microsoft/MicrosoftAzureCloudHSM/tree/main/IntegrationGuides).

## Available integration guides

Here are the integration guides available for securely connecting various applications and systems to Azure Cloud HSM.

- [Azure Cloud HSM ADCS Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20ADCS%20Integration%20Guide.pdf)
- [Azure Cloud HSM JCE Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20JCE%20Integration%20Guide.pdf)
- [Azure Cloud HSM OpenSSL Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20OpenSSL%20Integration%20Guide.pdf)
- [Azure Cloud HSM Oracle TDE Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20Oracle%20TDE%20Integration%20Guide.pdf)
- [Azure Cloud HSM PKCS11 Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20PKCS11%20Integration%20Guide.pdf)
- [Azure Cloud HSM SQL EKM Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20SQL%20EKM%20Integration%20Guide.pdf)
- [Azure Cloud HSM SSL TLS Offloading Integration Guide (PDF)](https://github.com/microsoft/MicrosoftAzureCloudHSM/blob/main/IntegrationGuides/Azure%20Cloud%20HSM%20SSL%20TLS%20Offloading%20Integration%20Guide.pdf)

## Next Steps
- [Azure Cloud HSM overview](overview.md)
- [Deploy Azure Cloud HSM using the Azure portal](quickstart-portal.md)
- [Deploy Azure Cloud HSM using Azure PowerShell](quickstart-powershell.md)
- [Azure Cloud HSM Onboarding Guide](onboarding-guide.md)