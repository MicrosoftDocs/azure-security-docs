---
title: How to migrate key workloads 
description: How to migrate key workloads
author: msmbaldwin
ms.author: mbaldwin
services: key-vault
ms.subservice: general
ms.topic: conceptual
ms.service: azure-key-vault
ms.date: 05/08/2025
---

# How to migrate key workloads

Azure Key Vault and Azure Managed HSM do not allow the export of keys, to protect the key material and ensure that the HSM properties of the keys can't be changed.

If you want a key to be highly portable, it is best to create it in a supported HSM and import it into Azure Key Vault or Azure Managed HSM.

> [!NOTE]
> The only exception to the no-export rule is when you create a key with a specific [key release policy](../keys/policy-grammar.md). This policy allows the key to be exported only to trusted confidential computing environments (secure enclaves) that you explicitly define. This limited export capability is designed for specific secure computing scenarios and is not the same as a general-purpose key export.

There are several scenarios that require the migration of key workloads:
- Switching security boundaries, such as when switching between subscriptions, resource groups, or owners.
- Moving regions due to compliance boundaries or risks in a given region.
- You are changing to a new offering, such as from Azure Key Vault to [Azure Managed HSM](../managed-hsm/overview.md), which offers greater security, isolation, and compliance than Key Vault Premium.

Below we discuss several methods for migrating workloads to use a new key, either into a new vault or into a new managed HSM.

## Azure Services using customer-managed key

For most workloads that use keys in Key Vault, the most effective way to migrate a key into a new location (a new managed HSM or new key vault in a different subscription or region) is to:

1. Create a new key in the new vault or managed HSM.
2. Grant your workload access to the new key by assigning the workload's managed identity to the appropriate RBAC role in [Azure Key Vault](rbac-guide.md) or [Azure Managed HSM](../managed-hsm/access-control.md).
1. Update the workload to use the new key as the customer managed encryption key.
1. Retain the old key until you no longer want the backups of the workload data that they key originally protected.

## Example: Migrating Azure Storage to a new customer-managed key

If you're using customer-managed keys with Azure Storage, you can migrate to a new key by following these steps:

1. Create the new key in your destination key vault or managed HSM.
2. Follow the instructions in [Configure customer-managed keys for an existing storage account](/azure/storage/common/customer-managed-keys-configure-existing-account) to update your storage account to use the new key.
3. Keep your previous customer-managed key available until the storage service is fully transitioned to the new key.
4. After confirming that all operations are working correctly with the new key, you can safely retire the previous key (but do not delete it if you need to access older backups).

This pattern applies to many Azure services that support customer-managed keys.

## Custom applications and client-side encryption

For client-side encryption or custom applications that directly encrypt data using the keys in Key Vault, the process is different:

1. Create the new key vault or managed HSM, and create a new key encryption key (KEK).
2. Re-encrypt any keys or data that encrypted by the old key using the new key. (If data is directly encrypted by the key in key vault, this may take some time, as all data must be read, decrypted, and encrypted with the new key. Use [envelope encryption](/azure/security/fundamentals/encryption-atrest#envelope-encryption-with-a-key-hierarchy) where possible to make such key rotations faster).

  When re-encrypting the data, we recommend a three-level key hierarchy, which will make KEK rotation easier in the future:
    1. The Key Encryption Key in Azure Key Vault or Managed HSM
    1. The Primary Key
    1. Data Encryption Keys derived from the Primary Key
1. Verify data after migration (and before deletion).
1. Do not delete old key/key vault until you no longer want the backups of data associated with it.

## Migrating tenant keys in Azure Information Protection

Migrating tenant keys in Azure Information Protection is referred to as "rekeying" or "rolling your key". [Customer-managed - AIP tenant key life cycle operations](/azure/information-protection/operations-customer-managed-tenant-key#rekey-your-tenant-key) has detailed instructions on how to perform this operation.

It is not safe to delete the old tenant key until you no longer need the content or documents protected with the old tenant key. If you want to migrate documents to be protected by the new key, you must:

1. Remove protection from the document protected with the old tenant key.
1. Apply protection again, which will use the new tenant key.

## Transitioning to HSM Platform 2

Azure Key Vault has updated its HSM platform to provide improved security with FIPS 140 Level 3 validation. All new keys and key versions are now created using HSM Platform 2 (except UK geo). You can check which HSM platform is protecting your key by looking at its [hsmPlatform](../keys/about-keys-details.md#key-attributes) attribute.

To transition your workloads to keys protected by HSM Platform 2:

1. **Create new keys on HSM Platform 2**
   - If you have a [key rotation policy](../keys/about-keys-details.md#key-rotation-policy-operations) configured, a new key version is automatically created at the next scheduled rotation.
   - If no rotation policy is configured, manually create a new key version which will automatically use HSM Platform 2.

1. **Rolling keys for different services**
   - **Customer Managed Keys (CMK)**:
     - If [autorotation for your keys](autorotation.md#key-autorotation) is enabled for your service, the new key is automatically applied when created.
     - If autorotate is not configured, update your service to use the new key manually using the service's key configuration settings.
   - **Azure Information Protection (AIP)**:
     - Refer to the [AIP tenant key migration section](#migrating-tenant-keys-in-azure-information-protection) for detailed migration steps.
   - **Custom Applications**:
     - Follow the [Custom applications guidance](#custom-applications-and-client-side-encryption) to ensure a smooth transition.

The benefits of transitioning to HSM Platform 2 include enhanced security compliance with FIPS 140 Level 3 validation. Since all new keys are automatically created on the latest platform, this transition mainly applies to updating existing workloads to use newer key versions.

## Managing Customer Keys for Microsoft 365

For organizations transitioning from HSM Platform 1, managing customer keys effectively is critical. Microsoft 365 provides robust tools and guidance for rolling or rotating customer-managed root keys and availability keys. Below are key resources to help you manage this process:

- [Roll or rotate a Customer Key or an availability key](/purview/customer-key-availability-key-roll): Learn how to roll customer-managed root keys or availability keys, including creating new versions or generating new keys.
- [Understand the availability key](/purview/customer-key-availability-key-understand): Detailed information about the availability key and its role in Microsoft 365 encryption.
- [Manage Customer Key for Microsoft 365](/purview/customer-key-manage): Comprehensive guidance on managing customer keys, including creating and assigning data encryption policies (DEPs).

### Key Considerations for HSM Platform 1 Retirement

As HSM Platform 1 is retired, ensure that you:

1. Roll or rotate your customer-managed root keys as needed to maintain compliance and security.
2. Update data encryption policies (DEPs) to reference new keys or key versions.
3. Follow best practices for key management, including minimizing permissions and monitoring key usage.

For more details, refer to the [Microsoft Purview Customer Key documentation](/purview/customer-key-overview).

## Next steps

- [About Azure Key Vault](overview.md)
- [Understanding auto-rotation in Azure Key Vault](autorotation.md)
- [About Azure Key Vault Managed HSM?](../managed-hsm/overview.md)
- [Key management is Azure](/azure/security/fundamentals/key-management)
