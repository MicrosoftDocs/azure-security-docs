---
ms.service: azure-key-vault
ms.topic: include
ms.date: 05/21/2026
---

The following operations are supported on key objects:

- **Create**: A client creates a key in the service. The service generates and stores the value of the key and doesn't release it to the client.
- **Import**: A client imports an existing key to the service. Asymmetric keys can be imported by using several different packaging methods within a JWK construct.
- **Update**: A client with sufficient permissions modifies the metadata (key attributes) associated with a previously stored key.
- **Delete**: A client with sufficient permissions deletes a key.
- **List**: A client lists all keys in the service.
- **List versions**: A client lists all versions of a given key.
- **Get**: A client retrieves the public parts of a given key. The get operation doesn't return the private portion of an asymmetric key, nor the key material of a symmetric key.
- **Backup**: Exports a key in a protected form.
- **Restore**: Imports a previously backed-up key.
- **Release**: Securely releases a key to authorized code running within a confidential compute environment. Requires an attestation that the Trusted Execution Environment (TEE) meets the requirements of the key's release_policy.

After a key is created, you can perform the following cryptographic operations with it:

- **Sign and Verify**: Strictly, this operation is "sign hash" or "verify hash", as the service doesn't hash content as part of signature creation. Applications should hash the data to be signed locally, then request that the service sign the hash. Verification of signed hashes is supported as a convenience for applications that might not have access to public key material; for best performance, verify operations should be performed locally.
- **Key Encryption / Wrapping**: A key stored in the service can protect another key, typically a symmetric content encryption key (CEK). When the key in the service is asymmetric, use key encryption (for example, RSA-OAEP; WRAPKEY/UNWRAPKEY are equivalent to ENCRYPT/DECRYPT). When the key in the service is symmetric, use key wrapping (for example, AES-KW). WRAPKEY is supported as a convenience for applications that might not have access to public key material; for best performance, wrap operations should be performed locally.
- **Encrypt and Decrypt**: A key stored in the service can encrypt or decrypt a single block of data. The size of the block is determined by the key type and the selected encryption algorithm. Encrypt is provided for convenience; for best performance, encrypt operations should be performed locally.

The use of distinct WRAPKEY/UNWRAPKEY operations (rather than always using ENCRYPT/DECRYPT) provides semantic and authorization separation, and consistency across key types.

The service doesn't support EXPORT operations. Once a key is provisioned, you can't extract it or modify its key material. To move a key, use the BACKUP and RESTORE operations to export or import the key in a protected form. Keys produced by BACKUP can't be used outside the service.

You can restrict any of the cryptographic operations on a per-key basis by using the `key_ops` property of the JWK object.

For more information about JWK objects, see [JSON Web Key (JWK)](https://tools.ietf.org/html/draft-ietf-jose-json-web-key-41).
