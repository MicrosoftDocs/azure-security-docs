---
title: Key types, algorithms, and operations - Azure Key Vault
description: Supported key types, algorithms, and operations (details).
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: keys
ms.topic: concept-article
ms.date: 03/06/2026
ms.author: mbaldwin
---

# Key types, algorithms, and operations

Key Vault supports two resource types: vaults and managed HSMs. Both resource types support various encryption keys. To see a summary of supported key types and protection types by each resource type, see [About keys](about-keys.md).

The following table shows a summary of key types and supported algorithms.

|Key types/sizes/curves| Encrypt/Decrypt<br>(Wrap/Unwrap) | Sign/Verify | 
| --- | --- | --- |
|EC-P256, EC-P256K, EC-P384, EC-P521|NA|ES256<br>ES256K<br>ES384<br>ES512|
|RSA 2K, 3K, 4K| RSA-OAEP-256<br>[Not recommended] RSA1_5<br>[Not recommended] RSA-OAEP|PS256<br>PS384<br>PS512<br>RS256<br>RS384<br>RS512<br>RSNULL| 
|oct 128-bit, 256-bit <br/>(Managed HSM only)| AES-KW<br>AES-GCM<br>AES-CBC| HS256<br>HS384<br>HS512| 
|||

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

##  Symmetric key algorithms (Managed HSM only)
The following algorithm identifiers are supported with oct-HSM keys.

### WRAPKEY/UNWRAPKEY, ENCRYPT/DECRYPT
- **AES-KW** - AES Key Wrap ([RFC3394](https://tools.ietf.org/html/rfc3394)).
- **AES-GCM** - AES encryption in Galois Counter Mode ([NIST SP 800-38d](https://csrc.nist.gov/publications/sp800)).
- **AES-CBC** - AES encryption in Cipher Block Chaining Mode ([NIST SP 800-38a](https://csrc.nist.gov/publications/sp800)).

When you use these algorithms with 256-bit keys, they're quantum-resistant according to the [The Commercial National Security Algorithm Suite 2.0 and Quantum Computing FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF).

### SIGN/VERIFY
- **HS256** - HMAC using SHA-256 hash function, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **HS384** - HMAC using SHA-384 hash function, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **HS512** - HMAC using SHA-512 hash function, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).

> [!NOTE] 
> The sign and verify operations algorithms must match the key type and size. Otherwise, the service returns a key size is incorrect error.

##  Key operations

Key Vault, including Managed HSM, supports the following operations on key objects:  

- **Create**: A client creates a key in Key Vault. Key Vault generates and stores the value of the key and doesn't release it to the client. You can create asymmetric keys in Key Vault.  
- **Import**: A client imports an existing key to Key Vault. You can import asymmetric keys to Key Vault by using several different packaging methods within a JWK construct. 
- **Update**: A client with sufficient permissions modifies the metadata (key attributes) associated with a key previously stored within Key Vault.  
- **Delete**: A client with sufficient permissions deletes a key from Key Vault.  
- **List**: A client lists all keys in a given Key Vault.  
- **List versions**: A client lists all versions of a given key in a given Key Vault.  
- **Get**: A client retrieves the public parts of a given key in a Key Vault.  
- **Backup**: Exports a key in a protected form.  
- **Restore**: Imports a previously backed up key.
- **Release**: Securely releases a key to authorized code running within a confidential compute environment. It requires an attestation that the Trusted Execution Environment (TEE) meets the requirements of the key’s release_policy.
- **Rotate**: Rotates an existing key by generating new version of the key (Key Vault only).

For more information, see [Key operations in the Key Vault REST API reference](/rest/api/keyvault).  

After you create a key in Key Vault, you can perform the following cryptographic operations by using the key:  

- **Sign and Verify**: Strictly, this operation is "sign hash" or "verify hash", as Key Vault doesn't support hashing of content as part of signature creation. Applications should hash the data to be signed locally, then request that Key Vault sign the hash.
Verification of signed hashes is supported as a convenience operation for applications that might not have access to [public] key material. For best application performance, VERIFY operations should be performed locally.  
- **Key Encryption / Wrapping**: You can use a key stored in Key Vault to protect another key, typically a symmetric content encryption key (CEK). When the key in Key Vault is asymmetric, use key encryption. For example, RSA-OAEP and the WRAPKEY/UNWRAPKEY operations are equivalent to ENCRYPT/DECRYPT. When the key in Key Vault is symmetric, use key wrapping. For example, AES-KW. The WRAPKEY operation is supported as a convenience for applications that might not have access to [public] key material. For best application performance, WRAPKEY operations should be performed locally.  
- **Encrypt and Decrypt**: You can use a key stored in Key Vault to encrypt or decrypt a single block of data. The size of the block is determined by the key type and selected encryption algorithm. The Encrypt operation is provided for convenience, for applications that might not have access to [public] key material. For best application performance, ENCRYPT operations should be performed locally.  

While WRAPKEY/UNWRAPKEY by using asymmetric keys might seem superfluous (as the operation is equivalent to ENCRYPT/DECRYPT), the use of distinct operations is important. The distinction provides semantic and authorization separation of these operations, and consistency when other key types are supported by the service.  

Key Vault doesn't support EXPORT operations. Once you provision a key in the system, you can't extract it or modify its key material. However, users of Key Vault might require their key for other use cases, such as after it is deleted. In this case, they can use the BACKUP and RESTORE operations to export or import the key in a protected form. You can't use keys created by the BACKUP operation outside Key Vault. Alternatively, you can use the IMPORT operation against multiple Key Vault instances.  

Users can restrict any of the cryptographic operations that Key Vault supports on a per-key basis by using the key_ops property of the JWK object.  

For more information about JWK objects, see [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41).  

##  Key rotation policy operations

Set key vault key auto-rotation by configuring the key auto-rotation policy. This feature is only available on Key Vault resources.

- **Get Rotation Policy**: Retrieve the rotation policy configuration.
- **Set Rotation Policy**: Set the rotation policy configuration.

## Key attributes

In addition to the key material, you can specify the following attributes. In a JSON request, you must include the `attributes` keyword and braces (`{` `}`) even if you don't specify any attributes.  

- *enabled*: boolean, optional, default is **true**. Specifies whether the key is enabled and usable for cryptographic operations. Use the *enabled* attribute with *nbf* and *exp*. When an operation occurs between *nbf* and *exp*, the operation is permitted only if *enabled* is set to **true**. Operations outside the *nbf* / *exp* window are automatically disallowed, except for [decrypt, release, unwrap, and verify](#date-time-controlled-operations).
- *nbf*: IntDate, optional, default is now. The *nbf* (not before) attribute identifies the time before which the key MUST NOT be used for cryptographic operations, except for [decrypt, release, unwrap, and verify](#date-time-controlled-operations). The processing of the *nbf* attribute requires that the current date/time MUST be after or equal to the not-before date/time listed in the *nbf* attribute. Key Vault MAY provide for some small leeway, normally no more than a few minutes, to account for clock skew. Its value MUST be a number containing an IntDate value.  
- *exp*: IntDate, optional, default is "forever". The *exp* (expiration time) attribute identifies the expiration time on or after which the key MUST NOT be used for cryptographic operation, except for [decrypt, release, unwrap, and verify](#date-time-controlled-operations). The processing of the *exp* attribute requires that the current date/time MUST be before the expiration date/time listed in the *exp* attribute. Key Vault MAY provide for some small leeway, typically no more than a few minutes, to account for clock skew. Its value MUST be a number containing an IntDate value.  

The following read-only attributes are included in any response that includes key attributes:  

- *created*: IntDate, optional. The *created* attribute indicates when this version of the key was created. The value is null for keys created before the addition of this attribute. Its value MUST be a number containing an IntDate value.  
- *updated*: IntDate, optional. The *updated* attribute indicates when this version of the key was updated. The value is null for keys that were last updated before the addition of this attribute. Its value MUST be a number containing an IntDate value.
- *hsmPlatform*: string, optional. The underlying HSM platform that protects a key.
    - A `hsmPlatform` value of `2` means the key is protected by the latest FIPS 140 Level 3 validated HSM platform.
    - A `hsmPlatform` value of `1` means the key is protected by the previous FIPS 140 Level 2 validated HSM platform.
    - A `hsmPlatform` value of `0` means the key is protected by a FIPS 140 Level 1 software cryptographic module.
    - If you don't set this value by using a Managed HSM pool, the key is protected by the latest FIPS 140 Level 3 validated HSM platform.
 
Keys are bound to the HSM in which you created them. You create and store new keys seamlessly in the new HSMs. While you can't migrate or transfer keys, new key versions are automatically in the new HSMs. For more information about how to migrate to a new key, see [How to migrate key workloads](../general/migrate-key-workloads.md).

For more information about IntDate and other data types, see [About keys, secrets, and certificates: Data types](../general/about-keys-secrets-certificates.md#data-types).

### Date-time controlled operations

Not-yet-valid and expired keys, outside the *nbf* / *exp* window, work for **decrypt**, **release**, **unwrap**, and **verify** operations (don't return 403, Forbidden). The rationale for using the not-yet-valid state is to allow a key to be tested before production use. The rationale for using the expired state is to allow recovery operations on data that you created when the key was valid. Also, you can disable access to a key by using Key Vault policies or by updating the *enabled* key attribute to **false**.

For more information about data types, see [Data types](../general/about-keys-secrets-certificates.md#data-types).

For more information about other possible attributes, see the [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41).

## Key tags

You can specify more application-specific metadata in the form of tags. Key Vault supports up to 15 tags, each of which can have a 256-character name and a 256-character value.  

> [!NOTE]
> If a caller has the *list* or *get* permission to a key, they can read the tags.

##  Key access control

Key Vault provides access control for keys at the Key Vault level, which acts as the container for keys. You can control access to keys by using either Key Vault [Azure role-based access control](../general/rbac-guide.md) (recommended) or the legacy [vault access policy](../general/assign-access-policy.md) permission model. Azure RBAC is the default and recommended authorization model. It has three predefined roles to manage keys: **Key Vault Crypto Officer**, **Key Vault Crypto User**, and **Key Vault Service Encryption User**. You can scope these roles to the subscription, resource group, or vault level. For more information, see [Azure RBAC vs. access policies](../general/rbac-access-policy.md).

Vault access policy permission model permissions:

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
- [About Managed HSM](../managed-hsm/overview.md)
- [About secrets](../secrets/about-secrets.md)
- [About certificates](../certificates/about-certificates.md)
- [Key Vault REST API overview](../general/about-keys-secrets-certificates.md)
- [Authentication, requests, and responses](../general/authentication-requests-and-responses.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
