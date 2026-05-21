---
title: Key types, algorithms, and operations - Azure Key Vault Managed HSM
description: Supported key types, algorithms, attributes, and operations in Azure Key Vault Managed HSM.
services: azure-key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: concept-article
ms.date: 05/21/2026
ms.author: mbaldwin
ai-usage: ai-assisted
---

# Key types, algorithms, and operations (Managed HSM)

Managed HSM supports RSA, Elliptic Curve, and symmetric (AES) keys. All keys are HSM-protected. For a high-level overview, see [About keys in Managed HSM](about-keys.md).

The following table summarizes supported key types and algorithms in Managed HSM.

| Key types / sizes / curves | Encrypt/Decrypt<br>(Wrap/Unwrap) | Sign/Verify |
| --- | --- | --- |
| EC-P256, EC-P256K, EC-P384, EC-P521 | NA | ES256<br>ES256K<br>ES384<br>ES512 |
| RSA 2K, 3K, 4K | RSA-OAEP-256<br>[Not recommended] RSA1_5<br>[Not recommended] RSA-OAEP | PS256<br>PS384<br>PS512<br>RS256<br>RS384<br>RS512<br>RSNULL |
| oct 128-bit, 192-bit, 256-bit | AES-KW<br>AES-GCM<br>AES-CBC | HS256<br>HS384<br>HS512 |

## EC algorithms

The following algorithm identifiers are supported with EC-HSM keys.

### Curve types

- **P-256** — The NIST curve P-256, defined at [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).
- **P-256K** — The SEC curve SECP256K1, defined at [SEC 2: Recommended Elliptic Curve Domain Parameters](https://www.secg.org/sec2-v2.pdf).
- **P-384** — The NIST curve P-384, defined at [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).
- **P-521** — The NIST curve P-521, defined at [DSS FIPS PUB 186-4](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.186-4.pdf).

### SIGN/VERIFY

- **ES256** — ECDSA for SHA-256 digests and keys created with curve P-256. Described at [RFC7518](https://tools.ietf.org/html/rfc7518).
- **ES256K** — ECDSA for SHA-256 digests and keys created with curve P-256K. Pending standardization.
- **ES384** — ECDSA for SHA-384 digests and keys created with curve P-384. Described at [RFC7518](https://tools.ietf.org/html/rfc7518).
- **ES512** — ECDSA for SHA-512 digests and keys created with curve P-521. Described at [RFC7518](https://tools.ietf.org/html/rfc7518).

## RSA algorithms

The following algorithm identifiers are supported with RSA-HSM keys.

### WRAPKEY/UNWRAPKEY, ENCRYPT/DECRYPT

- **RSA-OAEP-256** — RSAES using Optimal Asymmetric Encryption Padding with a hash function of SHA-256 and a mask generation function of MGF1 with SHA-256.
- [Not recommended] **RSA1_5** — RSAES-PKCS1-V1_5 [RFC3447] key encryption.
- [Not recommended] **RSA-OAEP** — RSAES using Optimal Asymmetric Encryption Padding (OAEP) [RFC3447], with the default parameters specified by RFC 3447 in Section A.2.1 (SHA-1 with MGF1/SHA-1).

> [!WARNING]
> Microsoft recommends using RSA_OAEP_256 or stronger algorithms for enhanced security.
>
> Microsoft does **not** recommend RSA_1_5 or RSA_OAEP, which are included solely for backward compatibility. Cryptographic standards no longer consider RSA with the PKCS#1 v1.5 padding scheme secure for encryption, and RSA_OAEP uses SHA-1, which has known collision problems.

### SIGN/VERIFY

- **PS256** — RSASSA-PSS using SHA-256 and MGF1 with SHA-256, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **PS384** — RSASSA-PSS using SHA-384 and MGF1 with SHA-384, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **PS512** — RSASSA-PSS using SHA-512 and MGF1 with SHA-512, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **RS256** — RSASSA-PKCS-v1_5 using SHA-256. The application-supplied digest must be computed using SHA-256 and must be 32 bytes in length.
- **RS384** — RSASSA-PKCS-v1_5 using SHA-384. The application-supplied digest must be computed using SHA-384 and must be 48 bytes in length.
- **RS512** — RSASSA-PKCS-v1_5 using SHA-512. The application-supplied digest must be computed using SHA-512 and must be 64 bytes in length.
- **RSNULL** — See [RFC2437](https://tools.ietf.org/html/rfc2437); a specialized use case to enable certain TLS scenarios.

> [!NOTE]
> RSA-PSS padding mode is recommended for better performance. The server constructs the DigestInfo for sign operations that algorithms RS256, RS384, and RS512 generate.

## Symmetric (AES) key algorithms

The following algorithm identifiers are supported with oct-HSM keys.

### WRAPKEY/UNWRAPKEY, ENCRYPT/DECRYPT

- **AES-KW** — AES Key Wrap ([RFC3394](https://tools.ietf.org/html/rfc3394)).
- **AES-GCM** — AES encryption in Galois Counter Mode ([NIST SP 800-38d](https://csrc.nist.gov/publications/sp800)).
- **AES-CBC** — AES encryption in Cipher Block Chaining Mode ([NIST SP 800-38a](https://csrc.nist.gov/publications/sp800)).

When used with 256-bit keys, these algorithms are quantum-resistant according to the [Commercial National Security Algorithm Suite 2.0 and Quantum Computing FAQ](https://media.defense.gov/2022/Sep/07/2003071836/-1/-1/0/CSI_CNSA_2.0_FAQ_.PDF).

### SIGN/VERIFY (HMAC)

- **HS256** — HMAC using SHA-256, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **HS384** — HMAC using SHA-384, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).
- **HS512** — HMAC using SHA-512, as described in [RFC7518](https://tools.ietf.org/html/rfc7518).

> [!NOTE]
> The sign and verify algorithm must match the key type and size. Otherwise, the service returns a key-size-incorrect error.

For throughput numbers per AES key size and operation, see [Managed HSM scaling guidance](scaling-guidance.md#aes-key-operations-number-of-operations-per-second-per-hsm-instance). To enforce a minimum AES key size with Azure Policy, see [Azure Policy for Managed HSM](azure-policy.md).

## Key operations

[!INCLUDE [Key operations](../includes/key-operations.md)]

For more information, see [Key operations in the Key Vault REST API reference](/rest/api/keyvault).

> [!NOTE]
> Managed HSM doesn't support automated key rotation policy operations; auto-rotation is available only on Key Vault resources. To rotate a key in Managed HSM, generate a new key version manually. For lifecycle guidance, see [How to migrate key workloads](../general/migrate-key-workloads.md).

## Key attributes

[!INCLUDE [Key attributes](../includes/key-attributes.md)]

Keys are bound to the Managed HSM instance in which they were created. You can't migrate or transfer a key between instances; to move a workload to a new key, see [How to migrate key workloads](../general/migrate-key-workloads.md). For more information about IntDate and other data types, see [Data types](../general/about-keys-secrets-certificates.md#data-types).

## Key tags

You can attach application-specific metadata in the form of tags. Up to 15 tags are supported per key, each with a 256-character name and a 256-character value.

> [!NOTE]
> If a caller has the *list* or *get* permission to a key, they can read its tags.

## Access control

Managed HSM uses a local, role-based access control (RBAC) model that's separate from Azure RBAC at the management plane. You assign built-in roles such as **Managed HSM Crypto User**, **Managed HSM Crypto Officer**, and **Managed HSM Administrator** at the HSM-instance or per-key scope. For details, see [Managed HSM local RBAC built-in roles](built-in-roles.md) and [Managed HSM access control](access-control.md).

## Related content

- [About keys in Managed HSM](about-keys.md)
- [Manage keys in Managed HSM](key-management.md)
- [Import HSM-protected keys to Managed HSM (BYOK)](hsm-protected-keys-byok.md)
- [Validate keys with key attestation](key-attestation.md)
- [Managed HSM scaling guidance](scaling-guidance.md)
- [Azure Policy for Managed HSM](azure-policy.md)
- [Managed HSM access control](access-control.md)
