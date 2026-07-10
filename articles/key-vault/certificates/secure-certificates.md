---
title: Secure your Azure Key Vault certificates
description: Learn how to secure Azure Key Vault certificates, with best practices specific to certificate management.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: best-practice
ms.custom: horz-security
ms.date: 07/10/2026
ai-usage: ai-assisted
# Customer intent: As a developer using Key Vault certificates, I want to implement certificate-specific security best practices.
---
# Secure your Azure Key Vault certificates

Azure Key Vault certificates manage X.509 certificates and their associated private keys for TLS/SSL, authentication, and code signing. This article provides security recommendations specific to certificate management.

> [!NOTE]
> This article focuses on security practices specific to Key Vault certificates. For comprehensive Key Vault security guidance including network security, identity and access management, and vault architecture, see [Secure your Azure Key Vault](../general/secure-key-vault.md).

## Certificate storage and format

Key Vault certificates combine X.509 certificates with their private keys and provide automated management capabilities:

- **Store certificates, not secrets**: Always use the Key Vault certificate object type rather than storing certificates as secrets. Certificate objects provide:
  - Automated lifecycle management
  - Integration with certificate authorities (CAs)
  - Automatic renewal capabilities
  - Built-in versioning

- **Use trusted certificate authorities**: Integrate with supported CAs for automated issuance and renewal:
  - DigiCert
  - GlobalSign
  - Other integrated CAs
  
  See [Integrating Key Vault with certificate authorities](how-to-integrate-certificate-authority.md).

- **Import external certificates properly**: When importing certificates from external sources:
  - Use PFX or PEM format
  - Include the full certificate chain
  - Protect private keys during import
  
  See [Import a certificate](tutorial-import-certificate.md).

For more information about certificates, see [About Azure Key Vault certificates](about-certificates.md).

## Certificate lifecycle management

Implement proper certificate lifecycle management to prevent expiration and outages:

- **Enable automatic renewal**: Configure automatic renewal for certificates issued by integrated CAs. For more information, see [Renew your Azure Key Vault certificates](overview-renew-certificate.md).

- **Set the renewal window ahead of expiration**: Configure the certificate to renew well before it expires. A common practice is to trigger renewal at about 80% of the certificate's lifetime. Adjust this value based on your CA's issuance latency and your change-control processes.

- **Monitor certificate expiration**: Use the Event Grid `Microsoft.KeyVault.CertificateNearExpiry` event, which fires 30 days before expiration, plus `Microsoft.KeyVault.CertificateExpired` and `Microsoft.KeyVault.CertificateNewVersionCreated` to track the full lifecycle. For more information, see [Azure Key Vault as Event Grid source](/azure/event-grid/event-schema-key-vault).

- **Maintain a certificate inventory**: Track all certificates, their purposes, owning applications, and expiration dates.

For more information about renewal, see [Tutorial: Configure certificate autorotation in Key Vault](tutorial-rotate-certificates.md).

## Certificate access control

Control who can access and manage certificates:

- **Separate certificate permissions**: Use Azure RBAC to grant specific certificate permissions:
  - **Key Vault Certificate User**: Read certificates and public keys.
  - **Key Vault Certificates Officer**: Manage certificate lifecycle (create, import, renew, delete).
  - **Key Vault Purge Operator**: Purge deleted certificates during the soft-delete retention period.

  For the full list of built-in roles, see [Azure built-in roles for Key Vault data plane operations](../general/rbac-guide.md#azure-built-in-roles-for-key-vault-data-plane-operations).

- **Limit administrative access**: Restrict certificate management operations to authorized personnel only

- **Use managed identities**: Applications should access certificates using managed identities rather than service principals with stored credentials

See [Provide access to Key Vault certificates with Azure RBAC](../general/rbac-guide.md).

## Certificate issuance policies

Configure certificate policies to enforce security requirements:

- **Set appropriate validity periods**:
  - **Publicly trusted TLS certificates**: Follow the current [CA/Browser Forum Baseline Requirements](https://cabforum.org/working-groups/server/baseline-requirements/documents/). Maximum public-TLS validity is reduced to 200 days (effective March 2026), and is scheduled to drop further (100 days in 2027, 47 days in 2029). Plan for shorter lifetimes and full automation of renewal.
  - **Internal certificates**: Set validity based on organizational policy, keeping lifetimes as short as your rotation automation supports.
  - **Code signing certificates**: Follow industry standards for the target platform.

- **Use strong key algorithms**:
  - RSA: 2048-bit minimum, 4096-bit for high-security scenarios
  - EC: P-256, P-384, or P-521 curves

- **Configure subject alternative names (SANs)**: Include all required DNS names and IP addresses

- **Set key usage extensions**: Specify appropriate key usage (Digital Signature, Key Encipherment) and extended key usage (Server Authentication, Client Authentication)

For more information about certificate policies, see [About Azure Key Vault certificate creation](create-certificate.md).

## Certificate monitoring and alerts

Track certificate operations and lifecycle events:

- **Enable diagnostic logging**: Log all certificate operations including:
  - Certificate creation and import
  - Certificate renewal attempts (success/failure)
  - Certificate access (get certificate, get private key)
  - Certificate deletion
  
  See [Azure Key Vault logging](../general/logging.md).

- **Configure expiration alerts**: Set up Azure Monitor alerts for:
  - Certificates expiring within 30 days
  - Failed renewal attempts
  - Certificate access by unauthorized identities
  
  See [Monitoring and alerting for Azure Key Vault](../general/alert.md).

- **Review certificate usage**: Regularly audit which applications and services are using each certificate

## Certificate export and backup

Protect certificate availability while maintaining security:

- **Control export operations**: Limit who can export certificates with private keys (exportable flag in certificate policy)

- **Enable soft delete**: Recover accidentally deleted certificates within the retention period (7-90 days). See [Azure Key Vault soft-delete overview](../general/soft-delete-overview.md)

- **Enable purge protection**: Prevent permanent deletion during the retention period. See [Purge protection](../general/soft-delete-overview.md#purge-protection)

- **Back up critical certificates**: Export and securely store backups of certificates for disaster recovery. See [Azure Key Vault backup](../general/backup.md)

- **Protect exported certificates**: When exporting certificates:
  - Use strong passwords for PFX files
  - Store exported files in secure locations
  - Delete temporary files after use
  - Audit export operations

## Certificate transparency and compliance

Maintain visibility into certificate issuance:

- **Understand Certificate Transparency (CT) behavior**: For publicly trusted certificates issued through Key Vault's integrated certificate authorities, the CA handles CT log submission - you don't enable it in Key Vault. Publicly trusted certificates and metadata leave the Azure boundary during enrollment and are logged by public CT logs per browser trust requirements. For more information, see the note on external CA and CT log handling in [About Azure Key Vault certificates](about-certificates.md#certificate-transparency).

- **Document certificate purposes**: Maintain records of:
  - Certificate purpose and owning application
  - Approval process for certificate issuance
  - Certificate renewal procedures

## Self-signed certificates

When using self-signed certificates for testing or internal purposes:

- **Limit to non-production environments**: Self-signed certificates should not be used in production for publicly accessible services

- **Set appropriate validity periods**: Use shorter validity periods for self-signed certificates (90 days or less)

- **Manage trust distribution**: Document how self-signed certificate trust is distributed to clients

- **Plan migration to CA-issued certificates**: Have a strategy to replace self-signed certificates with CA-issued certificates for production

For more information about self-signed certificates, see [Create a certificate with Key Vault](create-certificate.md).

## Related security articles

- [Secure your Azure Key Vault](../general/secure-key-vault.md): Comprehensive Key Vault security guidance.
- [Secure your Azure Key Vault keys](../keys/secure-keys.md): Security best practices for cryptographic keys.
- [Secure your Azure Key Vault secrets](../secrets/secure-secrets.md): Security best practices for secrets.

## Next steps

- [About Azure Key Vault certificates](about-certificates.md)
- [Renew your Azure Key Vault certificates](overview-renew-certificate.md)
- [Tutorial: Configure certificate autorotation in Key Vault](tutorial-rotate-certificates.md)
- [Integrate Key Vault with certificate authorities](how-to-integrate-certificate-authority.md)
