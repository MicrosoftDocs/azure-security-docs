---
title: Read a Key Vault secret from a .NET console app with certificate client credentials
description: Configure a certificate credential on a Microsoft Entra app registration and use ClientCertificateCredential from Azure.Identity to access Azure Key Vault from a .NET daemon.
author: garrodonnell
ms.author: godonnell
ms.date: 03/03/2026
ms.service: azure-key-vault
ms.subservice: general
ms.topic: how-to
ms.devlang: csharp
ai-usage: ai-assisted
#customer intent: As a developer I want to authenticate a .NET daemon app to Azure Key Vault by using a certificate credential so that I don't need to store client secrets.
---

# Read a Key Vault secret from a .NET console app with certificate client credentials

Use this guide when your app runs outside Azure where managed identity isn't available, or when a machine-to-machine daemon must authenticate without user interaction. Certificates are more secure than client secrets for production workloads. Use `ClientCertificateCredential` from the `Azure.Identity` library in your code.

The client credentials grant lets a confidential app authenticate as itself without a user. This guide walks you through configuring a certificate on your Microsoft Entra app registration and using it to retrieve secrets from Key Vault.

## Prerequisites

- An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- An app registration in Microsoft Entra ID.
- A certificate. Upload the public key (`.cer`) to the app registration and keep the private key (often `.pfx`) accessible to your app securely.

## Understand the OAuth 2.0 client credentials flow

Learn the protocol steps, what the token contains, and how app permissions are granted. For more information, see [Microsoft identity platform and the OAuth 2.0 client credentials flow](/entra/identity-platform/v2-oauth2-client-creds-grant-flow).

## Use a certificate as the application credential

Learn how to register certificates, how assertions are built, and thumbprint behavior. For more information, see [Application authentication certificate credentials](/entra/identity-platform/certificate-credentials).

## Add credentials in the app registration

Upload the `.cer` (public key) under **Certificates & secrets** for the app. For detailed steps, see [Add and manage application credentials in Microsoft Entra ID](/entra/identity-platform/how-to-add-credentials).

## Create a self-signed certificate for dev/test

If you need a certificate for development scenarios, use PowerShell to create and export a self-signed certificate. For steps, see [Create a self-signed public certificate to authenticate your application](/entra/identity-platform/howto-create-self-signed-certificate).

## Implement authentication in .NET

Use `ClientCertificateCredential` from `Azure.Identity` and the Key Vault Secrets client to fetch secrets. For the API reference, see [`ClientCertificateCredential` class](/dotnet/api/azure.identity.clientcertificatecredential). For sample code, see [Azure.Identity ClientCertificateCredential samples](https://github.com/Azure/azure-sdk-for-net/blob/main/sdk/identity/Azure.Identity/samples/ClientCertificateCredentialSamples.md).

After you implement the sample, acquire a token and read a known secret from Key Vault. Your app should authenticate by using the uploaded certificate, and the thumbprint should match.

> [!TIP]
> If you receive an AADSTS700027 or "key not found" error, ensure the exact certificate thumbprint is registered on the app. Re-upload the certificate if it was rotated. The [certificate credentials article](/entra/identity-platform/certificate-credentials) covers registration details.

## Next steps

- [Microsoft identity platform and the OAuth 2.0 client credentials flow](/entra/identity-platform/v2-oauth2-client-creds-grant-flow)
- [`ClientCertificateCredential` class](/dotnet/api/azure.identity.clientcertificatecredential)
- [Azure Key Vault developer's guide](developers-guide.md)