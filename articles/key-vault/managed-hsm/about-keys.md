---
title: About keys - Azure Key Vault Managed HSM
description: Overview of cryptographic keys in Azure Key Vault Managed HSM, including supported key types, sizes, compliance, and protection.
services: azure-key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: overview
ms.date: 05/21/2026
ms.author: mbaldwin
ai-usage: ai-assisted
---

# About keys in Managed HSM

Azure Key Vault Managed HSM is a fully managed, highly available, single-tenant cloud service that uses **FIPS 140-3 Level 3** validated hardware security modules to protect your cryptographic keys. All keys in a Managed HSM instance are HSM-protected; Managed HSM doesn't store software-protected keys.

Each Managed HSM instance is an isolated single-tenant pool with its own [security domain](security-domain.md), which provides complete cryptographic isolation from all other Managed HSM instances that share the same underlying hardware. The data-plane endpoint base URL for a Managed HSM instance is `https://<hsm-name>.managedhsm.azure.net`.

Cryptographic keys in Managed HSM are represented as JSON Web Key (JWK) objects, as defined by the following specifications:

- [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key)
- [JSON Web Encryption (JWE)](https://datatracker.ietf.org/doc/html/draft-jones-json-web-encryption)
- [JSON Web Algorithms (JWA)](https://datatracker.ietf.org/doc/html/draft-ietf-jose-json-web-algorithms)
- [JSON Web Signature (JWS)](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature)

The base JWK/JWA specifications are extended to enable key types unique to the Managed HSM implementation.

## Supported key types and sizes

| Key type | Sizes / curves |
|--|--|
| **RSA-HSM** — RSA key | 2,048-bit, 3,072-bit, 4,096-bit |
| **EC-HSM** — Elliptic Curve key | P-256, P-256K (secp256k1), P-384, P-521 |
| **oct-HSM** — Symmetric (AES) key | 128-bit, 192-bit, 256-bit |

You can generate keys directly in a Managed HSM instance, or import RSA, EC, or symmetric keys in soft form or by exporting them from a supported HSM device. To securely transport key material into Managed HSM, use the method described in the [BYOK (bring your own key) specification](../keys/byok-specification.md). For more information, see [Import HSM-protected keys to Managed HSM](hsm-protected-keys-byok.md).

For details on supported algorithms, operations, attributes, and tags, see [Key types, algorithms, and operations](about-keys-details.md).

## Compliance

All keys in Managed HSM are HSM-protected by **FIPS 140-3 Level 3** validated hardware. There is a single protection tier; Managed HSM doesn't expose multiple HSM platforms or a software-protected option.

## Quantum-resistant cryptography

"Quantum-resistant", "quantum-safe", and "post-quantum" cryptography are terms used to describe cryptographic algorithms believed to be resistant to cryptanalytic attacks from both classical and quantum computers. oct-HSM 256-bit keys used with the AES algorithms offered by Managed HSM are quantum-resistant. For more information, see [The Commercial National Security Algorithm Suite 2.0 and Quantum Computing FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/1/CSI_CNSA_2.0_FAQ_.PDF).

## Key attestation

Managed HSM can attest that a key was generated and resides inside an HSM that Microsoft operates. Asymmetric keys receive both public and private key attestation; symmetric (oct-HSM) keys receive private key attestation only. For more information, see [Validate Azure Managed HSM keys with key attestation](key-attestation.md).

## Usage scenarios

| When to use | Examples |
|--|--|
| Azure server-side data encryption with customer-managed keys | [Server-side encryption using customer-managed keys in Azure Key Vault](/azure/security/fundamentals/encryption-models#server-side-encryption-using-customer-managed-keys-in-azure-key-vault) |
| Client-side data encryption | [Client-Side Encryption with Azure Key Vault](/azure/storage/common/storage-client-side-encryption?tabs=dotnet) |
| Keyless TLS | Use the key [Client Libraries](../general/client-libraries.md#client-libraries-per-language-and-object) |

## Related content

- [About Managed HSM](overview.md)
- [Key types, algorithms, and operations](about-keys-details.md)
- [Manage keys in Managed HSM](key-management.md)
- [Import HSM-protected keys to Managed HSM](hsm-protected-keys-byok.md)
- [Key attestation](key-attestation.md)
- [Security domain overview](security-domain.md)
- [Managed HSM scaling guidance](scaling-guidance.md)
- [Key management in Azure](/azure/security/fundamentals/key-management)
