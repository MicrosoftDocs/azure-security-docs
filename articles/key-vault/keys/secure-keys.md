---
title: Secure your Azure Key Vault keys
description: Learn how to secure Azure Key Vault keys, with best practices specific to cryptographic key management.
author: msmbaldwin
tags: azure-key-vault
ms.service: azure-key-vault
ms.subservice: keys
ms.topic: conceptual
ms.custom: horz-security
ms.date: 11/10/2025
ms.author: mbaldwin
ai-usage: ai-assisted
# Customer intent: As a developer using Key Vault keys, I want to implement key-specific security best practices.
---
# Secure your Azure Key Vault keys

Azure Key Vault keys protect cryptographic keys used for encryption, digital signatures, and key wrapping operations. This article provides security recommendations specific to cryptographic key management.

> [!NOTE]
> This article focuses on security practices specific to Key Vault keys. For comprehensive Key Vault security guidance including network security, identity and access management, and vault architecture, see [Secure your Azure Key Vault](../general/secure-key-vault.md).

## Key types and protection levels

Azure Key Vault supports different key types with varying protection levels. Choose the appropriate key type based on your security requirements:

- **Software-protected keys (RSA, EC)**: Keys protected by FIPS 140-2 Level 1 validated software. Suitable for most applications requiring encryption and signing operations.

- **HSM-protected keys (RSA-HSM, EC-HSM)**: Keys protected by FIPS 140-2 Level 2 validated hardware security modules (HSMs). Recommended for high-security scenarios requiring hardware-backed key protection.

- **Managed HSM keys**: Keys in dedicated, single-tenant HSM pools with FIPS 140-2 Level 3 validated hardware. Required for the highest security and compliance requirements.

For more information about key types, see [About Azure Key Vault keys](about-keys.md).

## Key usage and operations

Restrict key operations to only those required for your application to minimize attack surface:

- **Limit key operations**: Grant only necessary permissions (encrypt, decrypt, sign, verify, wrapKey, unwrapKey)
- **Use appropriate key sizes**: 
  - RSA keys: Use 2048-bit minimum, 4096-bit for high-security scenarios
  - EC keys: Use P-256, P-384, or P-521 curves based on security requirements
- **Separate keys by purpose**: Use different keys for encryption vs. signing operations to limit impact if a key is compromised

For more information about key operations, see [Key operations in Key Vault](about-keys-details.md#key-operations).

## Key rotation and versioning

Implement regular key rotation to limit exposure from compromised keys:

- **Enable automatic key rotation**: Configure automatic rotation policies to rotate keys without application downtime. See [Configure key autorotation](how-to-configure-key-rotation.md)
- **Set rotation frequency**: Rotate encryption keys at least annually, or more frequently based on compliance requirements
- **Use key versioning**: Key Vault automatically versions keys, allowing seamless rotation without breaking existing encrypted data
- **Plan for re-encryption**: For long-term data, implement strategies to re-encrypt data with new key versions

For more information about rotation, see [Configure cryptographic key autorotation in Azure Key Vault](how-to-configure-key-rotation.md).

## Key backup and recovery

Protect against data loss by implementing proper backup and recovery procedures:

- **Enable soft delete**: Soft delete allows recovery of deleted keys within a retention period (7-90 days). See [Azure Key Vault soft-delete overview](../general/soft-delete-overview.md)
- **Enable purge protection**: Prevent permanent deletion of keys during the retention period. See [Purge protection](../general/soft-delete-overview.md#purge-protection)
- **Back up critical keys**: Export and securely store backups of keys that protect irreplaceable data. See [Azure Key Vault backup](../general/backup.md)
- **Document recovery procedures**: Maintain runbooks for key recovery scenarios

## Bring Your Own Key (BYOK)

When importing your own keys into Key Vault, follow security best practices:

- **Use secure key generation**: Generate keys in FIPS 140-2 Level 2 or higher HSMs
- **Protect keys during transfer**: Use Key Vault's BYOK process to securely transfer keys. See [Import HSM-protected keys to Key Vault (BYOK)](hsm-protected-keys-byok.md)
- **Validate key import**: Verify key attributes and permissions after import
- **Maintain key provenance**: Document the origin and transfer method of imported keys

For more information about BYOK, see [Import HSM-protected keys for Key Vault](hsm-protected-keys.md).

## Key release and attestation

For scenarios requiring key release to trusted environments:

- **Use key release policies**: Configure attestation-based release policies to control when keys can be released from Key Vault
- **Verify attestation**: Ensure requesting environments provide valid attestation before releasing keys
- **Audit key releases**: Monitor and log all key release operations

For more information about key release, see [Azure Key Vault key release](../keys/about-keys.md).

## Monitoring and auditing

Track key usage to detect unauthorized access or suspicious patterns:

- **Enable diagnostic logging**: Log all key operations for security analysis. See [Azure Key Vault logging](../general/logging.md)
- **Monitor key operations**: Track encrypt, decrypt, sign, and verify operations to establish baseline usage patterns
- **Set up alerts**: Configure Azure Monitor alerts for:
  - Unusual key access patterns
  - Failed key operations
  - Key deletions or modifications
  - Key approaching expiration
  
See [Monitoring and alerting for Azure Key Vault](../general/alert.md).

## Key expiration

Set expiration dates for keys when appropriate:

- **Set expiration for temporary keys**: Keys used for time-limited purposes should have expiration dates
- **Monitor expiring keys**: Use Event Grid notifications to alert before keys expire. See [Azure Key Vault as Event Grid source](/azure/event-grid/event-schema-key-vault)
- **Automate key renewal**: Implement automated processes to rotate keys before expiration

## Related security articles

- [Secure your Azure Key Vault](../general/secure-key-vault.md) - Comprehensive Key Vault security guidance
- [Secure your Azure Key Vault secrets](../secrets/secure-secrets.md) - Security best practices for secrets
- [Secure your Azure Key Vault certificates](../certificates/secure-certificates.md) - Security best practices for certificates

## Next steps

- [About Azure Key Vault keys](about-keys.md)
- [Configure cryptographic key autorotation in Azure Key Vault](how-to-configure-key-rotation.md)
- [Azure Key Vault developer's guide](../general/developers-guide.md)
