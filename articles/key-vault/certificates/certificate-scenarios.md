---
title: Get started with Key Vault certificates
description: Get started with Key Vault certificates management.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: get-started
ms.date: 04/10/2026

ms.author: mbaldwin
ms.custom: sfi-image-nochange
ai-usage: ai-assisted
---

# Get started with Key Vault certificates

This article helps you get started with certificate management in Key Vault. It covers the following scenarios:

- Creating your first Key Vault certificate
- Creating a certificate with a certificate authority (CA) that is partnered with Key Vault
- Creating a certificate with a CA that isn't partnered with Key Vault
- Importing a certificate

## Certificates are complex objects
Certificates are composed of three interrelated resources linked together as a Key Vault certificate; certificate metadata, a key, and a secret.


![Certificates are complex](../media/azure-key-vault.png)


## Creating your first Key Vault certificate  
Before a certificate can be created in Key Vault, the following prerequisite steps must be completed and a key vault must exist for this user or organization.  

**Step 1:** Certificate authority (CA) providers  
-   Onboarding as the IT admin, PKI admin, or anyone managing accounts with CAs, for a given company (for example, Contoso) is a prerequisite to using Key Vault certificates.  
    The following CAs are the current partnered providers with Key Vault. For more information, see [Partnered CA providers](./create-certificate.md#partnered-ca-providers).   
    -   DigiCert - Key Vault offers OV TLS/SSL certificates with DigiCert.  
    -   GlobalSign - Key Vault offers OV TLS/SSL certificates with GlobalSign.  

**Step 2:** An account admin for a CA provider creates credentials to be used by Key Vault to enroll, renew, and use TLS/SSL certificates.

**Step 3a:** A Contoso admin, along with a Contoso employee (Key Vault user) who owns certificates, depending on the CA, can get a certificate from the admin or directly from the account with the CA.  

- Begin an add credential operation to a key vault by [setting a certificate issuer](/rest/api/keyvault/certificates/set-certificate-issuer/set-certificate-issuer) resource. A certificate issuer is an entity represented in Azure Key Vault as a CertificateIssuer resource. It provides information about the source of a Key Vault certificate: issuer name, provider, credentials, and other administrative details.
  - For example, MyDigiCertIssuer  
    -   Provider  
    -   Credentials – CA account credentials. Each CA has its own specific data.  

    For more information on creating accounts with CA Providers, see [Integrating Key Vault with certificate authorities](./how-to-integrate-certificate-authority.md).  

**Step 3b:** Set up [certificate contacts](/rest/api/keyvault/certificates/set-certificate-contacts/set-certificate-contacts) for notifications. This is the contact for the Key Vault user. Key Vault doesn't enforce this step.  

> [!NOTE]
> This process, through **Step 3b**, is a onetime operation.  

## Creating a certificate with a CA partnered with Key Vault

![Create a certificate with a Key Vault partnered certificate authority](../media/certificate-authority-2.png)

**Step 4:** The following descriptions correspond to the green numbered steps in the preceding diagram.  
  (1) - In the diagram above, your application is creating a certificate which internally begins by creating a key in your key vault.  
  (2) - Key Vault sends a TLS/SSL Certificate Request to the CA.  
  (3) - Your application polls, in a loop and wait process, for your Key Vault for certificate completion. The certificate creation is complete when Key Vault receives the CA’s response with x509 certificate.  
  (4) - The CA responds to Key Vault's TLS/SSL Certificate Request with an X509 TLS/SSL Certificate.  
  (5) - Your new certificate creation completes with the merger of the X509 Certificate for the CA.  

  Key Vault user – creates a certificate by specifying a policy

  -   Repeat as needed  
  -   Policy constraints  
      -   X509 properties  
      -   Key properties  
      -   Provider reference -> for example, MyDigiCertIssuer  
      -   Renewal information -> for example, 90 days before expiry  

  - A certificate creation process is usually an asynchronous process and involves polling your key vault for the state of the create certificate operation.  
[Get certificate operation](/rest/api/keyvault/certificates/get-certificate-operation/get-certificate-operation)  
      -   Status: completed, failed with error information or, canceled  
      -   Because of the delay to create, a cancel operation can be initiated. The cancel may or may not be effective.  

### Network security and access policies associated with integrated CA
Key Vault service sends requests to the CA (outbound traffic). Therefore, it's fully compatible with firewall-enabled key vaults. Key Vault doesn't share access policies with the CA. The CA must be configured to accept signing requests independently. For more information, see [Integrating Key Vault with certificate authorities](./how-to-integrate-certificate-authority.md).

## Import a certificate  
Alternatively, you can import a certificate into Key Vault in PFX or PEM format.  

Importing a certificate requires a PEM or PFX file on disk that contains a private key. 
-   You must specify the vault name and certificate name (policy is optional).

-   PEM and PFX files contain attributes that Key Vault can parse and use to populate the certificate policy. If a certificate policy is already specified, Key Vault tries to match data from the PFX or PEM file.  

-   Once the import is final, subsequent operations use the new policy (new versions).  

-   If there are no further operations, the first thing Key Vault does is send an expiration notice. 

-   The user can also edit the policy, which is functional at the time of import but contains defaults where no information was specified at import. For example, no issuer info.  

### Formats of import we support
Azure Key Vault supports .pem and .pfx certificate files for importing certificates into a key vault.
The following import format is supported for PEM files: a single PEM-encoded certificate along with a PKCS#8-encoded, unencrypted key in the following format:

-----BEGIN CERTIFICATE-----

-----END CERTIFICATE-----

-----BEGIN PRIVATE KEY-----

-----END PRIVATE KEY-----

When you import a certificate, ensure that the key is included in the file itself. If you have the private key separately in a different format, you need to combine the key with the certificate. Some certificate authorities provide certificates in different formats, so before importing the certificate, make sure that it's in .pem or .pfx format. 


>[!NOTE]
>Ensure that no other meta data is present in the certificate file and that the private key not showing as encrypted.

### Formats of merge CSR we support

Azure Key Vault supports PKCS#8 encoded certificates with the following headers:

-----BEGIN CERTIFICATE-----

-----END CERTIFICATE-----

>[!NOTE]
> P7B (PKCS#7) signed certificate chains, commonly used by certificate authorities (CAs), are supported as long as they're base64-encoded. You can use [certutil -encode](/windows-server/administration/windows-commands/certutil#-encode) to convert to a supported format.

## Creating a certificate with a CA not partnered with Key Vault  
This method allows working with CAs other than Key Vault's partnered providers, meaning your organization can work with a CA of its choice.  

![Create a certificate with your own certificate authority](../media/certificate-authority-1.png)  

 The following step descriptions correspond to the green lettered steps in the preceding diagram.  

  1. In the diagram, your application is creating a certificate, which internally begins by creating a key in your key vault.  

  1. Key Vault returns a Certificate Signing Request (CSR) to your application.  

  1. Your application passes the CSR to your chosen CA.  

  1. Your chosen CA responds with an X.509 certificate.  

  1. Your application completes the new certificate creation with a merger of the X.509 certificate from your CA.
