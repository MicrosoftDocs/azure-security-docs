---
title: About keys - Azure Key Vault
description: Overview of Azure Key Vault REST interface and developer details for keys.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: keys
ms.topic: overview
ms.date: 09/06/2024
ms.author: mbaldwin
---

# About keys

Azure Key Vault provides two types of resources to store and manage cryptographic keys. Vaults support software-protected and HSM-protected (Hardware Security Module) keys. Managed HSMs only support HSM-protected keys.

|Resource type|Key protection methods|Data-plane endpoint base URL|
|--|--|--|
| **Vaults** | Software-protected and HSM-protected (HSM key types in Premium SKU) | https://{vault-name}.vault.azure.net |
| **Managed HSMs** | HSM-protected | https://{hsm-name}.managedhsm.azure.net |
||||

- **Vaults** - Vaults provide a low-cost, easy to deploy, multi-tenant, zone-resilient (where available), highly available key management solution suitable for most common cloud application scenarios.
- **Managed HSMs** - Managed HSM provides single-tenant, highly available HSMs to store and manage your cryptographic keys. Most suitable for applications and usage scenarios that handle high value keys. Also helps to meet most stringent security, compliance, and regulatory requirements. 

> [!NOTE]
> Vaults also allow you to store and manage several types of objects like secrets, certificates and storage account keys, in addition to cryptographic keys.

Cryptographic keys in Key Vault are represented as JSON Web Key [JWK] objects. The JavaScript Object Notation (JSON) and JavaScript Object Signing and Encryption (JOSE) specifications are:

- [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key)  
- [JSON Web Encryption (JWE)](https://datatracker.ietf.org/doc/html/draft-jones-json-web-encryption)  
- [JSON Web Algorithms (JWA)](https://datatracker.ietf.org/doc/html/draft-ietf-jose-json-web-algorithms)  
- [JSON Web Signature (JWS)](https://tools.ietf.org/html/draft-ietf-jose-json-web-signature) 

The base JWK/JWA specifications are also extended to enable key types unique to the Azure Key Vault and Managed HSM implementations. 

HSM Keys in vaults are protected by HSMs; Software keys aren't protected by HSMs.

- Keys stored in vaults benefit from robust protection using **[FIPS 140 validated HSM](/azure/key-vault/keys/about-keys#compliance)**. There are two distinct HSM platforms available: 1, which protects key versions with **FIPS 140-2 Level 2**, and 2, which protects keys with **FIPS 140-2 Level 3** HSMs depending on when the key was created. All new keys and key versions are now created using platform 2 (except UK geo). To determine which HSM Platform is protecting a key version, get it's [hsmPlatform](about-keys-details.md#key-attributes). 
- Managed HSM uses **FIPS 140-2 Level 3** validated HSM modules to protect your keys. Each HSM pool is an isolated single-tenant instance with its own [security domain](../managed-hsm/security-domain.md) providing complete cryptographic isolation from all other HSMs sharing the same hardware infrastructure. Managed HSM keys are protected in single-tenant HSM-pools. You can import an RSA, EC, and symmetric key, in soft form or by exporting from a supported HSM device. You can also generate keys in HSM pools. When you import HSM keys using the method described in the [BYOK (bring your own key) specification](../keys/byok-specification.md), it enables secure transportation key material into Managed HSM pools. 

For more information on geographical boundaries, see [Microsoft Azure Trust Center](https://azure.microsoft.com/support/trust-center/privacy/)

## Key types and protection methods

Key Vault supports RSA and EC keys. Managed HSM supports RSA, EC, and symmetric keys. 

### HSM-protected keys

|Key type|Vaults (Premium SKU only)|Managed HSMs|
|--|--|--|
|**EC-HSM**: Elliptic Curve key | Supported (P-256, P-384, P-521, secp256k1/P-256K)| Supported (P-256, secp256k1/P-256K, P-384, P-521)|
|**RSA-HSM**: RSA key|Supported (2048-bit, 3072-bit, 4096-bit)|Supported (2048-bit, 3072-bit, 4096-bit)|
|**oct-HSM**: Symmetric key|Not supported|Supported (128-bit, 192-bit, 256-bit)|
|||

### Software-protected keys

|Key type|Vaults|Managed HSMs|
|--|--|--|
**RSA**: "Software-protected" RSA key|Supported  (2048-bit, 3072-bit, 4096-bit)|Not supported
**EC**: "Software-protected" Elliptic Curve key |Supported  (P-256, P-384, P-521, secp256k1/P-256K)|Not supported
|||

### Compliance

|Key type and destination|Compliance|
|---|---|
|Software-protected (hsmPlatform 0) keys in vaults | FIPS 140-2 Level 1 |
|hsmPlatform 1 protected keys in vaults (Premium SKU)| FIPS 140-2 Level 2 |
|hsmPlatform 2 protected keys in vaults (Premium SKU)| FIPS 140-2 Level 3 |
|Keys in Managed HSM are always HSM protected | FIPS 140-2 Level 3 |
|||

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
- [About Managed HSM](../managed-hsm/overview.md)
- [About secrets](../secrets/about-secrets.md)
- [About certificates](../certificates/about-certificates.md)
- [Key Vault REST API overview](../general/about-keys-secrets-certificates.md)
- [Authentication, requests, and responses](../general/authentication-requests-and-responses.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
