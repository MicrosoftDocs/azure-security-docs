---
title: Key types, algorithms, and operations - Azure Key Vault
description: Supported key types, algorithms, attributes, and operations for vaults in Azure Key Vault.
services: key-vault

ms.service: azure-key-vault
ms.subservice: keys
ms.topic: concept-article
ms.date: 05/21/2026
ai-usage: ai-assisted
---

# Key types, algorithms, and operations

This article describes key types, algorithms, attributes, and operations for Azure Key Vault (vaults). For the equivalent reference for Managed HSM, see [Key types, algorithms, and operations (Managed HSM)](../managed-hsm/about-keys-details.md). For a summary of supported key types by resource, see [About keys](about-keys.md).

The following table shows a summary of key types and supported algorithms in Key Vault. For symmetric (oct-HSM / AES) algorithms, see [Key types, algorithms, and operations (Managed HSM)](../managed-hsm/about-keys-details.md).

|Key types/sizes/curves| Encrypt/Decrypt<br>(Wrap/Unwrap) | Sign/Verify | 
| --- | --- | --- |
|EC-P256, EC-P256K, EC-P384, EC-P521|NA|ES256<br>ES256K<br>ES384<br>ES512|
|RSA 2K, 3K, 4K| RSA-OAEP-256<br>[Not recommended] RSA1_5<br>[Not recommended] RSA-OAEP|PS256<br>PS384<br>PS512<br>RS256<br>RS384<br>RS512<br>RSNULL| 

##  EC algorithms
 The following algorithm identifiers are supported with EC-HSM keys.

### Curve types

- **P-256** - The NIST curve P-256, defined at [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).
- **P-256K** - The SEC curve SECP256K1, defined at [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf).
- **P-384** - The NIST curve P-384, defined at [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).
- **P-521** - The NIST curve P-521, defined at [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).

### SIGN/VERIFY

- **ES256** - ECDSA for SHA-256 digests and keys created with curve P-256. This algorithm is described at [RFC7518](https://tools.ietf.org/html/rfc7518).
- **ES256K** - ECDSA for SHA-256 digests and keys created with curve P-256K. This algorithm is pending standardization.
- **ES384** - ECDSA for SHA-384 digests and keys created with curve P-384. This algorithm is described at [RFC7518](https://tools.ietf.org/html/rfc7518).
- **ES512** - ECDSA for SHA-512 digests and keys created with curve P-521. This algorithm is described at [RFC7518](https://tools.ietf.org/html/rfc7518).

##  RSA algorithms  
 The following algorithm identifiers are supported with RSA and RSA-HSM keys.  

### WRAPKEY/UNWRAPKEY, ENCRYPT/DECRYPT

-  **RSA-OAEP-256** – RSAES using Optimal Asymmetric Encryption Padding with a hash function of SHA-256 and a mask generation function of MGF1 with SHA-256.
- [Not recommended] **RSA1_5** - RSAES-PKCS1-V1_5 [RFC3447] key encryption.  
- [Not recommended] **RSA-OAEP** - RSAES using Optimal Asymmetric Encryption Padding (OAEP) [RFC3447], with the default parameters specified by RFC 3447 in Section A.2.1. Those default parameters are using a hash function of SHA-1 and a mask generation function of MGF1 with SHA-1.
  
> [!WARNING]
> Microsoft recommends using RSA_OAEP_256 or stronger algorithms for enhanced security. 
>
> Microsoft does **not** recommend RSA_1_5 and RSA_OAEP, which are included solely for backwards compatibility. Cryptographic standards no longer consider RSA with the PKCS#1 v1.5 padding scheme secure for encryption, while RSA_OAEP utilizes SHA1, which has known collision problems. 

### SIGN/VERIFY

- **PS256** - RSASSA-PSS using SHA-256 and MGF1 with SHA-256, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **PS384** - RSASSA-PSS using SHA-384 and MGF1 with SHA-384, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **PS512** - RSASSA-PSS using SHA-512 and MGF1 with SHA-512, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **RS256** - RSASSA-PKCS-v1_5 using SHA-256. The application supplied digest value must be computed using SHA-256 and must be 32 bytes in length.  
- **RS384** - RSASSA-PKCS-v1_5 using SHA-384. The application supplied digest value must be computed using SHA-384 and must be 48 bytes in length.  
- **RS512** - RSASSA-PKCS-v1_5 using SHA-512. The application supplied digest value must be computed using SHA-512 and must be 64 bytes in length.  
- **RSNULL** - See [RFC2437](https://tools.ietf.org/html/rfc2437), a specialized use-case to enable certain TLS scenarios.  

> [!NOTE]
> RSA-PSS padding mode is recommended for better performance. The server constructs the DigestInfo for Sign operations that algorithms RS256, RS384, and RS512 generate.

##  Symmetric key algorithms

Symmetric (oct-HSM / AES) keys aren't supported in Key Vault. For supported AES algorithms (AES-KW, AES-GCM, AES-CBC) and HMAC sign/verify (HS256/384/512), see [Key types, algorithms, and operations (Managed HSM)](../managed-hsm/about-keys-details.md#symmetric-aes-key-algorithms).

##  Key operations

[!INCLUDE [Key operations](../includes/key-operations.md)]

In addition to the operations listed above, Key Vault (vaults) supports:

- **Rotate**: Rotates an existing key by generating a new version of the key. This operation is available only on Key Vault resources (not on Managed HSM).

For more information, see [Key operations in the Key Vault REST API reference](/rest/api/keyvault).

##  Key rotation policy operations

Set key auto-rotation by configuring the key auto-rotation policy. This feature is only available on Key Vault resources.

- **Get Rotation Policy**: Retrieve the rotation policy configuration.
- **Set Rotation Policy**: Set the rotation policy configuration.

## Key attributes

[!INCLUDE [Key attributes](../includes/key-attributes.md)]

In addition to the attributes listed above, Key Vault keys expose:

- *hsmPlatform*: string, optional. The underlying HSM platform that protects a key.
    - A `hsmPlatform` value of `2` means the key is protected by the latest FIPS 140-3 Level 3 validated HSM platform.
    - A `hsmPlatform` value of `1` means the key is protected by the previous FIPS 140-2 Level 2 validated HSM platform.
    - A `hsmPlatform` value of `0` means the key is protected by a FIPS 140-2 Level 1 software cryptographic module.

Keys are bound to the HSM in which you created them. New keys and key versions are seamlessly created on the latest HSM platform. You can't migrate or transfer existing keys between platforms; for guidance on moving workloads to a new key, see [How to migrate key workloads](../general/migrate-key-workloads.md).

For more information about IntDate and other data types, see [Data types](../general/about-keys-secrets-certificates.md#data-types).

## Key tags

You can specify more application-specific metadata in the form of tags. Key Vault supports up to 15 tags, each of which can have a 256-character name and a 256-character value.  

> [!NOTE]
> If a caller has the *list* or *get* permission to a key, they can read the tags.

##  Key access control

Key Vault provides access control for keys at the Key Vault level, which acts as the container for keys. You can control access to keys by using either Key Vault [Azure role-based access control](../general/rbac-guide.md) (recommended) or the legacy [vault access policy](../general/assign-access-policy.md) permission model. Azure RBAC is the default and recommended authorization model. It has three predefined roles to manage keys: **Key Vault Crypto Officer**, **Key Vault Crypto User**, and **Key Vault Service Encryption User**. You can scope these roles to the subscription, resource group, or vault level. For more information, see [Azure RBAC vs. access policies](../general/rbac-access-policy.md).

Vault access policy permission model permissions (legacy):

- Permissions for key management operations
  - *get*: Read the public part of a key, plus its attributes
  - *list*: List the keys or versions of a key stored in a key vault
  - *update*: Update the attributes for a key
  - *create*: Create new keys
  - *import*: Import a key to a key vault
  - *delete*: Delete the key object
  - *recover*: Recover a deleted key
  - *backup*: Back up a key in a key vault
  - *restore*: Restore a backed up key to a key vault

- Permissions for cryptographic operations
  - *decrypt*: Use the key to unprotect a sequence of bytes
  - *encrypt*: Use the key to protect an arbitrary sequence of bytes
  - *unwrapKey*: Use the key to unprotect wrapped symmetric keys
  - *wrapKey*: Use the key to protect a symmetric key
  - *verify*: Use the key to verify digests  
  - *sign*: Use the key to sign digests
    
- Permissions for privileged operations
  - *purge*: Purge (permanently delete) a deleted key
  - *release*: Release a key to a confidential compute environment, which matches the `release_policy` of the key

- Permissions for rotation policy operations
  - *rotate*: Rotate an existing key by generating new version of the key (Key Vault only) 
  - *get rotation policy*: Retrieve rotation policy configuration
  - *set rotation policy*: Set rotation policy configuration

For more information about working with keys, see [Key operations in the Key Vault REST API reference](/rest/api/keyvault). 

## Next steps
- [About Key Vault](../general/overview.md)
- [About keys in Managed HSM](../managed-hsm/about-keys.md)
- [Key types, algorithms, and operations (Managed HSM)](../managed-hsm/about-keys-details.md)
- [About secrets](../secrets/about-secrets.md)
- [About certificates](../certificates/about-certificates.md)
- [Key Vault REST API overview](../general/about-keys-secrets-certificates.md)
- [Authentication, requests, and responses](../general/authentication-requests-and-responses.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
