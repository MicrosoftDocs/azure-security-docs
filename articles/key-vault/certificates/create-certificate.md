---
title: Certificate creation methods  
description: Learn about different options to create or import a Key Vault certificate in Azure Key Vault. There are several ways to create a Key Vault certificate.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: concept-article
ms.date: 04/10/2026

ms.author: mbaldwin

---

# Certificate creation methods

 A Key Vault certificate can be either created or imported into a key vault. When a Key Vault certificate is created, the private key is created inside the key vault and never exposed to the certificate owner. The following are ways to create a certificate in Key Vault:  

- **Create a self-signed certificate:** Create a public-private key pair and associate it with a certificate. The certificate will be signed by its own key.  

- **Create a new certificate manually:** Create a public-private key pair and generate an X.509 certificate signing request. The signing request can be signed by your registration authority or certification authority. The signed X.509 certificate can be merged with the pending key pair to complete the Key Vault certificate in Key Vault. Although this method requires more steps, it does provide you with greater security because the private key is created in and restricted to Key Vault.

![Create a certificate with your own certificate authority](../media/certificate-authority-1.png)  

The following descriptions correspond to the green lettered steps in the preceding diagram.

1. In the diagram, your application is creating a certificate, which internally begins by creating a key in your key vault.
2. Key Vault returns to your application a Certificate Signing Request (CSR)
3. Your application passes the CSR to your chosen CA.
4. Your chosen CA responds with an X509 Certificate.
5. Your application completes the new certificate creation with a merger of the X509 Certificate from your CA.

- **Create a certificate with a known issuer provider:** This method requires you to do a one-time task of creating an issuer object. Once an issuer object is created in your key vault, its name can be referenced in the policy of the Key Vault certificate. A request to create such a Key Vault certificate will create a key pair in the vault and communicate with the issuer provider service using the information in the referenced issuer object to get an X.509 certificate. The X.509 certificate is retrieved from the issuer service and is merged with the key pair to complete the Key Vault certificate creation.  

![Create a certificate with a Key Vault partnered certificate authority](../media/certificate-authority-2.png)  

The following descriptions correspond to the green lettered steps in the preceding diagram.

1. In the diagram, your application is creating a certificate, which internally begins by creating a key in your key vault.
2. Key Vault sends a TLS/SSL Certificate Request to the CA.
3. Your application polls, in a loop and wait process, for your Key Vault for certificate completion. The certificate creation is complete when Key Vault receives the CA’s response with an X.509 certificate.
4. The CA responds to Key Vault's TLS/SSL Certificate Request with a TLS/SSL X.509 certificate.
5. Your new certificate creation completes with the merger of the TLS/SSL X.509 certificate for the CA.

## Asynchronous process

Key Vault certificate creation is an asynchronous process. This operation creates a Key Vault certificate request and returns an HTTP status code of 202 (Accepted). The status of the request can be tracked by polling the pending object created by this operation. The full URI of the pending object is returned in the LOCATION header.  

When a request to create a Key Vault certificate completes, the status of the pending object changes to "completed" from "in progress", and a new version of the Key Vault certificate is created. This becomes the current version.  

## First creation
 When a Key Vault certificate is created for the first time, an addressable key and secret are also created with the same name as the certificate. If the name is already in use, then the operation fails with an HTTP status code of 409 (conflict).
 The addressable key and secret get their attributes from the Key Vault certificate attributes. The addressable key and secret created this way are marked as managed keys and secrets, whose lifetime is managed by Key Vault. Managed keys and secrets are read-only. Note: If a Key Vault certificate expires or is disabled, the corresponding key and secret become inoperable.  

 If this is the first operation to create a Key Vault certificate, a policy is required.  A policy can also be supplied with successive create operations to replace the policy resource. If a policy isn't supplied, then the policy resource on the service is used to create a next version of the Key Vault certificate. While a request to create a next version is in progress, the current Key Vault certificate, and corresponding addressable key and secret, remain unchanged.  

## Self-issued certificate
 To create a self-issued certificate, set the issuer name as "Self" in the certificate policy as shown in following snippet from certificate policy.  

```  
"issuer": {  
       "name": "Self"  
    }  

```  

 If the issuer name isn't specified, then the issuer name is set to "Unknown". When issuer is "Unknown", the certificate owner will have to manually get an X.509 certificate from the issuer of their choice, then merge the public X.509 certificate with the key vault certificate pending object to complete the certificate creation.

```  
"issuer": {  
       "name": "Unknown"  
    }  

```  

## Partnered CA providers

Certificate creation can be completed manually or using a "Self" issuer. Key Vault also partners with certain issuer providers to simplify the creation of certificates. The following types of certificates can be ordered for key vault with these partner issuer providers.  

|Provider|Certificate type|Configuration setup  
|--------------|----------------------|------------------|
|DigiCert|Key Vault offers OV or EV SSL certificates with DigiCert| [Integration Guide](./how-to-integrate-certificate-authority.md)
|GlobalSign|Key Vault offers OV or EV SSL certificates with GlobalSign| [Integration Guide](https://support.globalsign.com/code-signing/Code-Signing-certificate-setup-in-Azure-Key-vault)

 A certificate issuer is an entity represented in Azure Key Vault as a CertificateIssuer resource. It provides information about the source of a Key Vault certificate: issuer name, provider, credentials, and other administrative details.

When an order is placed with the issuer provider, it may honor or override the X.509 certificate extensions and certificate validity period based on the type of certificate.  

 Authorization: Requires the certificates/create permission.

## Next steps

 - How-to guide to create certificates in Key Vault using [Portal](./quick-create-portal.md), [Azure CLI](./quick-create-cli.md), [Azure PowerShell](./quick-create-powershell.md)
 - [Monitor and manage certificate creation](create-certificate-scenarios.md)
