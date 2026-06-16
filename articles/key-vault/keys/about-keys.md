---
title: About keys - Azure Key Vault
description: Overview of Azure Key Vault REST interface and developer details for keys.
services: key-vault

ms.service: azure-key-vault
ms.subservice: keys
ms.topic: overview
ms.date: 05/21/2026
ai-usage: ai-assisted
---

# About keys

Azure Key Vault provides two types of resources to store and manage cryptographic keys. Vaults support software-protected and HSM-protected (Hardware Security Module) keys. Managed HSMs only support HSM-protected keys.

|Resource type|Key protection methods|Data-plane endpoint base URL|
|--|--|--|
| **Vaults** | Software-protected and HSM-protected (HSM key types in Premium SKU) | `https://<vault-name>.vault.azure.net` |
| **Managed HSMs** | HSM-protected | `https://<hsm-name>.managedhsm.azure.net` |

- **Vaults** - Vaults provide a low-cost, easy to deploy, multi-tenant, zone-resilient (where available), highly available key management solution suitable for most common cloud application scenarios.
- **Managed HSMs** - Managed HSM provides single-tenant, highly available HSMs to store and manage your cryptographic keys. Most suitable for applications and usage scenarios that handle high value keys. Also helps to meet most stringent security, compliance, and regulatory requirements.

> [!NOTE]
> This article describes keys in Key Vault (vaults). For details specific to Managed HSM, see [About keys in Managed HSM](../managed-hsm/about-keys.md) and [Key types, algorithms, and operations (Managed HSM)](../managed-hsm/about-keys-details.md).

> [!NOTE]
> Vaults also allow you to store and manage several types of objects like secrets, certificates and storage account keys, in addition to cryptographic keys.

Cryptographic keys in Key Vault are represented as JSON Web Key [JWK] objects. The JavaScript Object Notation (JSON) and JavaScript Object Signing and Encryption (JOSE) specifications are:

- [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key)  
- [JSON Web Encryption (JWE)](https://datatracker.ietf.org/doc/html/draft-jones-json-web-encryption)  
- [JSON Web Algorithms (JWA)](https://datatracker.ietf.org/doc/html/draft-ietf-jose-json-web-algorithms)  
- [JSON Web Signature (JWS)](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature) 

The base JWK/JWA specifications are also extended to enable key types unique to the Azure Key Vault implementation.

HSM Keys in vaults are protected by HSMs; Software keys aren't protected by HSMs.

Keys stored in vaults benefit from robust protection using **[FIPS 140 validated HSM](/azure/key-vault/keys/about-keys#compliance)**. There are two distinct HSM platforms available: HSM Platform 1, which protects key versions with **FIPS 140-2 Level 2**, and HSM Platform 2, which protects keys with **FIPS 140-3 Level 3** HSMs depending on when the key was created. All new keys and key versions are now created using HSM Platform 2. To determine which HSM platform is protecting a key version, get its [hsmPlatform](about-keys-details.md#key-attributes) attribute.

For more information on geographical boundaries, see [Microsoft Azure Trust Center](https://azure.microsoft.com/support/trust-center/privacy/).

## Key types and protection methods

Key Vault Premium and Standard support RSA and EC keys. For symmetric (oct-HSM / AES) keys, use [Managed HSM](../managed-hsm/about-keys.md).

### HSM-protected keys (Key Vault Premium)

|Key type|Sizes / curves|
|--|--|
|**EC-HSM**: Elliptic Curve key | P-256, P-384, P-521, secp256k1/P-256K |
|**RSA-HSM**: RSA key | 2048-bit, 3072-bit, 4096-bit |

### Software-protected keys (Key Vault Standard and Premium)

|Key type|Sizes / curves|
|--|--|
|**RSA**: Software-protected RSA key | 2048-bit, 3072-bit, 4096-bit |
|**EC**: Software-protected Elliptic Curve key | P-256, P-384, P-521, secp256k1/P-256K |

For Managed HSM support, see [About keys in Managed HSM](../managed-hsm/about-keys.md).

### Compliance

|Key type and destination|Compliance|
|---|---|
|Software-protected (HSM Platform 0) keys in vaults | FIPS 140-2 Level 1 |
|HSM Platform 1 protected keys in vaults (Premium SKU)| FIPS 140-2 Level 2 |
|HSM Platform 2 protected keys in vaults (Premium SKU)| FIPS 140-3 Level 3 |

### Quantum-resistant, Quantum-safe, or Post-quantum Cryptography

"Quantum-resistant", "quantum-safe", and "post-quantum" cryptography are all terms used to describe cryptographic algorithms believed to be resistant to cryptanalytic attacks from both classical and quantum computers. oct-HSM 256-bit keys used with the AES algorithms offered by Managed HSM are quantum-resistant. For more information, see [The Commercial National Security Algorithm Suite 2.0 and Quantum Computing FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/1/CSI_CNSA_2.0_FAQ_.PDF).

See [Key types, algorithms, and operations](about-keys-details.md) for details about each key type, algorithms, operations, attributes, and tags.

## Usage Scenarios

| When to use | Examples |
|--------------|-------------|
| Azure server-side data encryption for integrated resource providers with customer-managed keys | - [Server-side encryption using customer-managed keys in Azure Key Vault](/azure/security/fundamentals/encryption-models#server-side-encryption-using-customer-managed-keys-in-azure-key-vault) |
| Client-side data encryption | - [Client-Side Encryption with Azure Key Vault](/azure/storage/common/storage-client-side-encryption?tabs=dotnet)|
| Keyless TLS | - Use key [Client Libraries](../general/client-libraries.md#client-libraries-per-language-and-object) |

## Next steps
- [Key management in Azure](/azure/security/fundamentals/key-management)
- [About Key Vault](../general/overview.md)
- [About keys in Managed HSM](../managed-hsm/about-keys.md)
- [About secrets](../secrets/about-secrets.md)
- [About certificates](../certificates/about-certificates.md)
- [Key Vault REST API overview](../general/about-keys-secrets-certificates.md)
- [Authentication, requests, and responses](../general/authentication-requests-and-responses.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
