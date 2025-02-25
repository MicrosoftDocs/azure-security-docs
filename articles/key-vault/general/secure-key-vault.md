---
title: Secure your Azure Key Vault
description: An overview of best practices and guidance on how to secure your Azure Key Vault deployment
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: conceptual
ms.date: 01/25/2025
ms.author: mbaldwin
#Customer intent: As a key vault administrator, I want to learn how to secure my key vaults
---

# Secure your Azure Key Vault

Azure Key Vault protects cryptographic keys, certificates (and the private keys associated with the certificates), and secrets (such as connection strings and passwords) in the cloud. When storing sensitive and business-critical data, however, you must take steps to maximize the security of your vaults and the data stored in them.

This article provides guidance on how to best secure your Azure Key Vault deployment.

## Key vault architecture

- **Use one Key Vault per application, region, and environment**: Create separate Key Vaults for development, preproduction, and production environments to reduce the impact of breaches.

    Key vaults define security boundaries for stored secrets. Grouping secrets into the same vault increases the blast radius of a security event because attacks might be able to access secrets across concerns. To mitigate access across concerns, consider what secrets a specific application should have access to, and then separate your key vaults based on this delineation. Separating key vaults by application is the most common boundary. Security boundaries, however, can be more granular for large applications, for example, per group of related services.

- **Use one Key Vault per tenant in multitenant solutions**: For multitenant SaaS solutions, use a separate Key Vault for each tenant to maintain data isolation. More details are in [Multitenancy and Azure Key Vault](/azure/architecture/guide/multitenant/service/key-vault).

## Object Storage in Key Vault

- **Do not use Key Vault as a data storage to store customer configurations or service configurations**:  Services should use [Azure Storage with encryption at rest](/azure/storage/common/storage-service-encryption) or [Azure configuration manager](/mem/configmgr/core/understand/introduction). Storage is more performant for such scenarios.

- **Do not store certificates (customer or service owned) as secrets**:  Service-owned certificates should be stored as Key Vault certificates and configured for autorotation. For more information, see [Azure key vault: Certificates](../certificates/about-certificates.md).

    - **Customer content (excluding secrets and certificates) should not be stored in Key Vault**: Key Vault is not a data store and not built to scale like a data store. Instead use a proper data store like [Cosmos DB](/azure/cosmos-db/introduction) or [Azure Storage](/azure/storage/common/storage-introduction). Customers have the option of BYOK (Bring Your Own Key) for encryption at rest. This key can be stored in Azure Key Vault to encrypt the data in Azure Storage.

## Posture and vulnerability management

- **Use Azure Policy to enforce configuration**: Configure Azure Policy to audit and enforce secure configurations for Azure Key Vault and set up alerts for deviations from policy. See [Azure Policy Regulatory Compliance controls for Azure Key Vault](../security-controls-policy.md).

## Network Security

These network security features are listed from most restricted to least restricted capabilities. Pick the configuration that best suits your organization's use case.

- **Disable public network access and use Private Endpoints only**: Deploy Azure Private Link to establish a private access point from a virtual network to Azure Key Vault and prevent exposure to the public internet. For implementation steps, see [Integrate Key Vault with Azure Private Link](private-link-service.md).

    - Some customer scenarios require trusted Microsoft services to bypass the firewall, in such cases the vault might need to be configured to allow Trusted Microsoft Services. For full details, see [Network security: Key Vault Firewall Enabled (Trusted Services Only)](network-security.md?#key-vault-firewall-enabled-trusted-services-only).

- **Enable Key Vault Firewall**: Limit access to public static IP addresses or your virtual networks. For full details, see [Key Vault network security: firewall settings](network-security.md#firewall-settings).

    - Some customer scenarios require trusted Microsoft services to bypass the firewall, in such cases the vault might need to be configured to allow Trusted Microsoft Services.

- **Use Network Security Perimeter (preview)**: Define a logical network isolation boundary for PaaS resources (for example, Azure Key Vault, Azure Storage and SQL Database) that are deployed outside your organization’s virtual network perimeter and/or public static IP addresses. For full details, see [Network security: Network Security Perimeter (preview)](network-security.md#network-security-perimeter-preview)

  - "publicNetworkAccess": "SecuredByPerimeter" overrides "Allow trusted Microsoft services to bypass the firewall", meaning that some scenarios that require that trust will not work.

## Identity Management

- **Enable managed identities**: Use Azure managed identities for all app and service connections to Azure Key Vault to eliminate hard-coded credentials. Managed identities help secure authentication while removing the need for explicit credentials. Learn more in [Azure Key Vault authentication](authentication.md).

- **Use role-based access control**: Use Azure Role-Based Access Control (RBAC) to manage access to the Azure Key Vault. For more information, see [Azure RBAC for Key Vault data plane operations](./rbac-guide.md).

    - **Do not use legacy access policies**: Do not use legacy access policies, as they lack support for Privileged Identity Management (PIM). Azure RBAC mitigates potential unauthorized Key Vault access risks. See [Azure role-based access control (Azure RBAC) vs. access policies (legacy)](rbac-access-policy.md).

- **Assign just-in-time (JIT) privileged roles**: Use Azure Privileged Identity Management (PIM) to assign eligible JIT RBAC roles for administrators and operators of Key Vault. See [Privileged Identity Management (PIM) overview](/azure/active-directory/privileged-identity-management/pim-configure) for details.

    - **Require approvals for privileged role activation**: Add an extra layer of security to prevent unauthorized access by ensuring that at least one approver is required to activate JIT roles. See [Configure Microsoft Entra role settings in Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-how-to-change-default-settings).
    - **Enforce multi-factor authentication for role activation**: Require MFA to activate JIT roles for operators and administrators. See [Microsoft Entra multifactor authentication](/entra/identity/authentication/concept-mfa-howitworks).

- **Enable Microsoft Entra Conditional Access Policies**: Key Vault supports Microsoft Entra Conditional Access policies to apply access controls based on conditions such as user location or device. For more information, see [Conditional Access overview](/azure/active-directory/conditional-access/overview).

- **Apply the principle of least privilege**: Limit the number of users with administrative roles and ensure users are granted only the minimum permissions required for their role. See [Enhance security with the principle of least privilege](/entra/identity-platform/secure-least-privileged-access)

## Data Protection

- **Turn on soft delete**: Ensure that soft delete is enabled so that deleted Key Vault objects can be recovered within a 7 to 90-day retention period. See [Azure Key Vault soft-delete overview](soft-delete-overview.md).

- **Turn on purge protection**: Enable purge protection to protect against accidental or malicious deletion of Key Vault objects even after soft delete is enabled. See [Azure Key Vault soft-delete overview: Purge Protection](soft-delete-overview.md#purge-protection)

## Logging and Threat Detection

- **Enable audit logging**: Key Vault logging saves information about operations performed on the vault. For full details, see [Key Vault logging](logging.md).

- **Enable Microsoft Defender for Key Vault**: Enable Microsoft Defender for Key Vault to monitor for and alert on suspicious activity. For details, see [Microsoft Defender for Key Vault introduction](/azure/defender-for-cloud/defender-for-key-vault-introduction).

- **Enable log alerts for security events**: Set up alerts to be notified when critical events are logged, such as access failures or secret deletions. See [Monitoring and alerting for Azure Key Vault](alert.md).

- **Monitor and alert**: Integrate Key Vault with Event Grid to receive notifications on changes to keys, certificates, or secrets. For details, see [Monitoring Key Vault with Azure Event Grid](event-grid-overview.md).

## Backup and Recovery

- **Enable native backup for Azure Key Vault**: Configure and use the Azure Key Vault native backup feature to back up secrets, keys, and certificates, ensuring recoverability. See [Azure Key Vault backup](backup.md).

- **Ensure backups for secrets that can't be recreated**: Back up Key Vault objects (like encryption keys) that can't be recreated from other sources. See [Azure Key Vault backup](backup.md).

- **Test backup and recovery procedures**: To verify the effectiveness of backup processes, regularly test the restoration of Key Vault secrets, keys, and certificates. See [Azure Key Vault backup](backup.md).

## Next steps

- [Azure Key Vault security baseline](/security/benchmark/azure/baselines/key-vault-security-baseline)
- [Virtual network service endpoints for Azure Key Vault](overview-vnet-service-endpoints.md)
- [Azure RBAC: Built-in roles](/azure/role-based-access-control/built-in-roles)
