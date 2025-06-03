---
title: Comprehensive Guide to Key Management in Azure Cloud HSM
description: Learn best practices and recommendations for key management in Azure Cloud HSM, including storage limits, key wrapping security, and caching strategies.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin

#customer intent: As a security administrator, I need to understand key management in Azure Cloud HSM so that I can ensure optimal performance, security, and efficiency.
---

# Key management in Azure Cloud HSM

Effective key management is critical for optimizing the performance, security, and efficiency of Azure Cloud HSM. This article provides best practices and recommendations for handling key storage limits, key wrapping security, key attributes, and caching strategies.

## Manage the key storage limit

Hardware security modules (HSMs) have limits on the maximum number of tokens and session keys that can be stored at a single time. For details on these limits, refer to [Azure Cloud HSM service limits](service-limits.md).

To avoid exceeding Azure Cloud HSM service limits, consider employing one or more of the following strategies for efficient key management:

- **Key rotation**: Rotate keys frequently to ensure that older keys are replaced and space is freed for new ones. Frequent rotation helps keep the HSM within storage limits while maintaining security.
- **Key hierarchy**: Use a key hierarchy in which you use primary keys to encrypt other keys. This hierarchy reduces the number of keys that need to be stored in the HSM directly.
- **Key sharing and reuse**: For applications with multiple sessions or tokens, consider sharing keys or reusing them to reduce the total number stored.
- **Key deletion**: After you no longer need a key (for example, after a session ends), ensure that it's securely deleted to free up space for new keys.

> [!NOTE]
> Wait 24 hours after you create a key to ensure that synchronization and backups within your Azure Cloud HSM deployment are complete.

## Manage key wrapping

You use the `EXTRACTABLE` attribute in Azure Cloud HSM to mark keys as either extractable or nonextractable. By default, HSM keys are set as extractable. You can export extractable keys from the HSM through key wrapping, which encrypts the keys. The keys then require unwrapping via the same wrapping key before use.

In contrast, nonextractable keys can't be exported from Azure Cloud HSM under any circumstances. After you set keys as nonextractable, there's no way to change them to extractable. It's crucial to carefully consider whether you need your keys to be extractable and set the key attribute accordingly.

If your application necessitates key wrapping, we recommend that you use *trusted key wrapping*. This approach restricts HSM users to wrap and unwrap only keys that an admin explicitly designated as trusted:

- `EXTRACTABLE=0`: Keys created with the `EXTRACTABLE` attribute set to `0` can't be exported, except as masked objects. They're ideal for keys that you don't ever want to leave the HSM.

- `WRAP_WITH_TRUSTED=1`: By default, extractable keys created through the Azure Cloud HSM SDK use trusted key wrapping. However, the specification for PKCS#11 sets `WRAP_WITH_TRUSTED` to `CK_FALSE (0)` by default. Without trusted key wrapping, a cryptography user can export a key's private material without any authorization. Anyone who has access to a client application and uses those keys can export them in plain text.

### Key attribute support for Azure Cloud HSM providers

| Azure Cloud HSM provider | Default attributes for private and symmetric keys | Supports EXTRACTABLE=0 | Supports configuring WRAP_WITH_TRUSTED within the provider | Default WRAP_WITH_TRUSTED value |
|--------------------------|----------------------------------------------|------------------------|---------------------------------------------|-------------------------|
| `azcloudhsm_util`         | `EXTRACTABLE`, `!WRAP_WITH_TRUSTED`             | Yes                    | Yes                                         | `1` (`0` can be set via parameters) |
| PKCS#11                 | `EXTRACTABLE`, `!WRAP_WITH_TRUSTED`             | Yes                    | Yes                                         | `0` (specified in the PKCS#11 specification, but can be set to `1` in the API) |
| CNG/KSP               | `EXTRACTABLE`, `!WRAP_WITH_TRUSTED`             | No                     | No                                          | `1` |
| OpenSSL engine          | `EXTRACTABLE`, `!WRAP_WITH_TRUSTED`             | No                     | No                                          | `1` |
| JCE                     | `!EXTRACTABLE`, `!PERSISTANT`, `!WRAP_WITH_TRUSTED` | Yes                    | No                                          | `1` |

Keys that you create by using the Cryptography API: Next Generation (CNG) provider are always set as `EXTRACTABLE`. You can use the `CavImportKey.exe` tool to import keys from other Azure Cloud HSM tools into the CNG provider via their key handles. This import creates a key in the key storage provider (KSP) from existing key handles.

For *certificate authorities that use CNG or KSP as a provider*, all keys generated within the CNG provider are still marked as extractable. If a key is marked as `TRUSTED` on the HSM (like a user-generated key encryption key), it might be used to extract the private key plaintext of these keys, even after the `WRAP_WITH_TRUSTED` change.

In such cases, we recommend using `azcloudhsm_util` to generate the key if you don't want it to be marked as `EXTRACTABLE`. We recommend that you use `getAttribute` to verify that the generated keys have the desired attributes.

## Employ key attributes to manage key permissions

Use key attributes to manage key capabilities like permissions. When you're generating a key, use key attributes to specify permissions that permit or restrict specific operations for that key. We recommend that you generate keys with only the necessary attributes for their intended purpose.

For instance, an *Advanced Encryption Standard (AES) key that's used for encryption should not have the ability to wrap keys outside the HSM*. For more information on attributes for the Azure Cloud HSM SDK, see the [integration guides](integration-guides.md).

## Optimize latency by caching key objects

To reduce latency, consider caching key objects whenever possible. In searches for keys, each HSM in your Azure Cloud HSM cluster is queried. This operation is costly and doesn't scale efficiently. The method for key searches depends on the provider:

- For PKCS#11, key searches use the `C_FindObjects` API.
- For Java Cryptography Extension (JCE), key searches use the `KeyStore` value.

For optimal performance, we recommend that you use key-finding commands (such as `findKey` and `key list`) *only once during application startup*. Store the returned key object in application memory. When you need this key object later, retrieve it from the cache instead of querying for it with each operation. Querying for it incurs significant performance overhead.

## Related content

- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Network security in Azure Cloud HSM](network-security.md)
- [User management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)
