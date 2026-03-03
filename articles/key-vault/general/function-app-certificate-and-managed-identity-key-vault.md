---
title: Load a private certificate or use managed identity to access Key Vault from Azure Functions
description: Load a private certificate into a function app by using WEBSITE_LOAD_CERTIFICATES and use managed identity as the recommended approach for Key Vault access.
author: garrodonnell
ms.author: godonnell
ms.date: 03/03/2026
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ai-usage: ai-assisted
#customer intent: As a developer I want to load a private certificate into my function app or use managed identity to access Key Vault without storing secrets in configuration.
---

# Load a private certificate or use managed identity to access Key Vault from Azure Functions

The recommended approach for Azure Functions is managed identity for service-to-service authentication, including Key Vault access. However, you might still need to load a private certificate for outbound mTLS or third-party integrations. This guide covers both the certificate-loading steps and the managed-identity-based alternative for Key Vault.

## Prerequisites

- An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An Azure function app.

## Load a private certificate into your function app

Azure Functions runs on App Service, so follow the App Service certificate steps:

1. Upload the certificate under **Certificates** (Bring your own certificates).

1. Expose the certificate to code by setting `WEBSITE_LOAD_CERTIFICATES` to the certificate thumbprint, a comma-separated list of thumbprints, or `*` for all certificates.

1. Access the certificate at runtime:
   - **Windows** - The certificate is in the certificate store at `CurrentUser\My`.
   - **Linux** - The certificate is available as files under `/var/ssl/` paths.

For the complete walkthrough, see [Use TLS/SSL certificates in your application code in Azure App Service](/azure/app-service/configure-ssl-certificate-in-code).

> [!TIP]
> After you change `WEBSITE_LOAD_CERTIFICATES`, restart the app. For Linux plans or containers, use the documented file paths rather than the Windows certificate store.

## Access Key Vault from functions by using managed identity

Set up your function app to use identities instead of secrets or certificates for Key Vault and other Azure services. For the complete walkthrough, see [Create a function app that connects to Azure services using identities instead of secrets](/azure/azure-functions/functions-identity-based-connections-tutorial).

> [!TIP]
> If you only need Key Vault secrets in app settings, consider Key Vault references with an identity to avoid handling certificates or secret values in configuration.

## Validate your setup

- **Certificate path** - Confirm that your function can open the certificate (Windows store or Linux file path) after you set `WEBSITE_LOAD_CERTIFICATES`.
- **Managed identity path** - Use `DefaultAzureCredential` to read a known secret from Key Vault. The [tutorial](/azure/azure-functions/functions-identity-based-connections-tutorial) covers identity setup and role assignment.

> [!NOTE]
> If a certificate isn't found, verify the exact thumbprint in `WEBSITE_LOAD_CERTIFICATES`. On Linux, confirm the expected files exist under `/var/ssl`. If Key Vault access is denied, grant the function's identity a Key Vault data-plane role such as **Key Vault Secrets User** on the specific vault, then retry.

## Next steps

- [Use TLS/SSL certificates in your application code in Azure App Service](/azure/app-service/configure-ssl-certificate-in-code)
- [Create a function app that connects to Azure services using identities instead of secrets](/azure/azure-functions/functions-identity-based-connections-tutorial)
- [Azure Key Vault developer's guide](developers-guide.md)