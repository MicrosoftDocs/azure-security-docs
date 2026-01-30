---
title: Secure your Azure Key Vault
description: Learn how to secure Azure Key Vault, with best practices for protecting your deployment.
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.custom: horz-security
ms.topic: article
ms.date: 01/30/2026
ms.author: mbaldwin
ai-usage: ai-assisted
#CustomerIntent: As a key vault administrator, I want to learn how to secure my key vaults
---

# Secure your Azure Key Vault

Azure Key Vault protects cryptographic keys, certificates (and the private keys associated with the certificates), and secrets (such as connection strings and passwords) in the cloud. When storing sensitive and business-critical data, however, you must take steps to maximize the security of your vaults and the data stored in them.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

This article provides security recommendations to help protect your Azure Key Vault deployment.

## Service-specific security

Azure Key Vault has unique security considerations related to vault architecture and appropriate use of the service for storing cryptographic materials.

### Key vault architecture

- **Use one Key Vault per application, region, and environment**: Create separate Key Vaults for development, preproduction, and production environments to reduce the impact of breaches.

    Key vaults define security boundaries for stored secrets. Grouping secrets into the same vault increases the *blast radius* of a security event because attacks might be able to access secrets across concerns. To mitigate access across concerns, consider what secrets a specific application should have access to, and then separate your key vaults based on this delineation. Separating key vaults by application is the most common boundary. Security boundaries, however, can be more granular for large applications, for example, per group of related services.

- **Use one Key Vault per tenant in multitenant solutions**: For multitenant SaaS solutions, use a separate Key Vault for each tenant to maintain data isolation. This is the recommended approach for secure isolation of customer data and workloads. See [Multitenancy and Azure Key Vault](/azure/architecture/guide/multitenant/service/key-vault).

### Object Storage in Key Vault

- **Do not use Key Vault as a data storage to store customer configurations or service configurations**:  Services should use [Azure Storage with encryption at rest](/azure/storage/common/storage-service-encryption) or [Azure configuration manager](/mem/configmgr/core/understand/introduction). Storage is more performant for such scenarios.

- **Do not store certificates (customer or service owned) as secrets**:  Service-owned certificates should be stored as Key Vault certificates and configured for autorotation. For more information, see [Azure key vault: Certificates](../certificates/about-certificates.md) and [Understanding autorotation in Azure Key Vault](autorotation.md).

    - **Customer content (excluding secrets and certificates) should not be stored in Key Vault**: Key Vault is not a data store and not built to scale like a data store. Instead use a proper data store like [Cosmos DB](/azure/cosmos-db/introduction) or [Azure Storage](/azure/storage/common/storage-introduction). Customers have the option of BYOK (Bring Your Own Key) for encryption at rest. This key can be stored in Azure Key Vault to encrypt the data in Azure Storage.

## Network Security

Reducing network exposure is critical to protecting Azure Key Vault from unauthorized access. Configure network restrictions based on your organization's requirements and use case.

These network security features are listed from most restricted to least restricted capabilities. Pick the configuration that best suits your organization's use case.

- **Disable public network access and use Private Endpoints only**: Deploy Azure Private Link to establish a private access point from a virtual network to Azure Key Vault and prevent exposure to the public internet. For implementation steps, see [Integrate Key Vault with Azure Private Link](private-link-service.md).

    - Some customer scenarios require trusted Microsoft services to bypass the firewall, in such cases the vault might need to be configured to allow Trusted Microsoft Services. For full details, see [Network security: Key Vault Firewall Enabled (Trusted Services Only)](network-security.md?#key-vault-firewall-enabled-trusted-services-only).

- **Enable Key Vault Firewall**: Limit access to public static IP addresses or your virtual networks. For full details, see [Key Vault network security: firewall settings](network-security.md#firewall-settings).

    - Some customer scenarios require trusted Microsoft services to bypass the firewall, in such cases the vault might need to be configured to allow Trusted Microsoft Services.

- **Use Network Security Perimeter**: Define a logical network isolation boundary for PaaS resources (for example, Azure Key Vault, Azure Storage and SQL Database) that are deployed outside your organization's virtual network perimeter and/or public static IP addresses. For full details, see [Network security: Network Security Perimeter)](network-security.md#network-security-perimeter )

  - "publicNetworkAccess": "SecuredByPerimeter" overrides "Allow trusted Microsoft services to bypass the firewall", meaning that some scenarios that require that trust will not work.

## TLS and HTTPS

Azure Key Vault supports TLS 1.2 and 1.3 protocol versions to ensure secure communication between clients and the service.

- **Enforce TLS version control**: The Key Vault front end (data plane) is a multi-tenant server where key vaults from different customers can share the same public IP address. To achieve isolation, each HTTP request is authenticated and authorized independently. The HTTPS protocol allows clients to participate in TLS negotiation, and clients can enforce the TLS version to ensure the entire connection uses the corresponding level of protection. See [Key Vault logging](logging.md) for sample Kusto queries to monitor TLS versions used by clients.

## Identity and access management

Azure Key Vault uses Microsoft Entra ID for authentication. Access is controlled through two interfaces: the control plane (for managing Key Vault itself) and the data plane (for working with keys, secrets, and certificates). For details on the access model and endpoints, see [Azure RBAC for Key Vault data plane operations](rbac-guide.md).

- **Enable managed identities**: Use Azure managed identities for all app and service connections to Azure Key Vault to eliminate hard-coded credentials. Managed identities help secure authentication while removing the need for explicit credentials. For authentication methods and scenarios, see [Azure Key Vault authentication](authentication.md).

- **Use role-based access control**: Use Azure Role-Based Access Control (RBAC) to manage access to the Azure Key Vault. For more information, see [Azure RBAC for Key Vault data plane operations](./rbac-guide.md).

    - **Do not use legacy access policies**: Legacy access policies have known security vulnerabilities and lack support for Privileged Identity Management (PIM), and should not be used for critical data and workloads. Azure RBAC mitigates potential unauthorized Key Vault access risks. See [Azure role-based access control (Azure RBAC) vs. access policies (legacy)](rbac-access-policy.md).

    > [!IMPORTANT]
    > RBAC permission model allows vault-level role assignments for persistent access and eligible (JIT) assignments for privileged operations. Object-scope assignments only support read operations; administrative operations like network access control, monitoring, and object management require vault-level permissions. For secure isolation across application teams, use one Key Vault per application.

- **Assign just-in-time (JIT) privileged roles**: Use Azure Privileged Identity Management (PIM) to assign eligible JIT Azure RBAC roles for administrators and operators of Key Vault. See [Privileged Identity Management (PIM) overview](/entra/id-governance/privileged-identity-management/pim-configure) for details.

    - **Require approvals for privileged role activation**: Add an extra layer of security to prevent unauthorized access by ensuring that at least one approver is required to activate JIT roles. See [Configure Microsoft Entra role settings in Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-how-to-change-default-settings).
    - **Enforce multi-factor authentication for role activation**: Require MFA to activate JIT roles for operators and administrators. See [Microsoft Entra multifactor authentication](/entra/identity/authentication/concept-mfa-howitworks).

- **Enable Microsoft Entra Conditional Access Policies**: Key Vault supports Microsoft Entra Conditional Access policies to apply access controls based on conditions such as user location or device. For more information, see [Conditional Access overview](/entra/identity/conditional-access/overview).

- **Apply the principle of least privilege**: Limit the number of users with administrative roles and ensure users are granted only the minimum permissions required for their role. See [Enhance security with the principle of least privilege](/entra/identity-platform/secure-least-privileged-access)

## Data Protection

Protecting data stored in Azure Key Vault requires enabling soft delete, purge protection, and implementing automated rotation of cryptographic materials.

- **Turn on soft delete**: Ensure that soft delete is enabled so that deleted Key Vault objects can be recovered within a 7 to 90-day retention period. See [Azure Key Vault soft-delete overview](soft-delete-overview.md).

- **Turn on purge protection**: Enable purge protection to protect against accidental or malicious deletion of Key Vault objects even after soft delete is enabled. See [Azure Key Vault soft-delete overview: Purge Protection](soft-delete-overview.md#purge-protection)

- **Implement autorotation for cryptographic assets**: Configure automatic rotation of keys, secrets, and certificates to minimize the risk of compromise and ensure compliance with security policies. Regular rotation of cryptographic materials is a critical security practice. See [Understanding autorotation in Azure Key Vault](autorotation.md), [Configure key autorotation](../keys/how-to-configure-key-rotation.md), [Configure certificate autorotation](../certificates/tutorial-rotate-certificates.md), [Automate secret rotation for resources with one set of authentication credentials](../secrets/tutorial-rotation.md), and [Automate secret rotation for resources with two sets of authentication credentials](../secrets/tutorial-rotation-dual.md).

## Compliance and governance

Regular compliance audits and governance policies ensure your Key Vault deployment adheres to security standards and organizational requirements.

- **Use Azure Policy to enforce configuration**: Configure Azure Policy to audit and enforce secure configurations for Azure Key Vault and set up alerts for deviations from policy. See [Azure Policy Regulatory Compliance controls for Azure Key Vault](../security-controls-policy.md).

## Logging and Threat Detection

Comprehensive logging and monitoring enable detection of suspicious activities and compliance with audit requirements.

- **Enable audit logging**: Key Vault logging saves information about operations performed on the vault. For full details, see [Key Vault logging](logging.md).

- **Enable Microsoft Defender for Key Vault**: Enable Microsoft Defender for Key Vault to monitor for and alert on suspicious activity. For details, see [Microsoft Defender for Key Vault introduction](/azure/defender-for-cloud/defender-for-key-vault-introduction).

- **Enable log alerts for security events**: Set up alerts to be notified when critical events are logged, such as access failures or secret deletions. See [Monitoring and alerting for Azure Key Vault](alert.md).

- **Monitor and alert**: Integrate Key Vault with Event Grid to receive notifications on changes to keys, certificates, or secrets. For details, see [Monitoring Key Vault with Azure Event Grid](event-grid-overview.md).

## Backup and Recovery

Regular backups ensure business continuity and protect against data loss from accidental or malicious deletion.

- **Enable native backup for Azure Key Vault**: Configure and use the Azure Key Vault native backup feature to back up secrets, keys, and certificates, ensuring recoverability. See [Azure Key Vault backup](backup.md).

- **Ensure backups for secrets that can't be recreated**: Back up Key Vault objects (like encryption keys) that can't be recreated from other sources. See [Azure Key Vault backup](backup.md).

- **Test backup and recovery procedures**: To verify the effectiveness of backup processes, regularly test the restoration of Key Vault secrets, keys, and certificates. See [Azure Key Vault backup](backup.md).

## Related security articles

For security best practices specific to keys, secrets, and certificates, see:

- [Secure your Azure Key Vault keys](../keys/secure-keys.md) - Key-specific security best practices including rotation, HSM protection, and BYOK
- [Secure your Azure Key Vault secrets](../secrets/secure-secrets.md) - Secrets-specific security best practices including rotation, caching, and monitoring
- [Secure your Azure Key Vault certificates](../certificates/secure-certificates.md) - Certificate-specific security best practices including lifecycle management, renewal, and CA integration

## Next steps

- [Azure Key Vault security baseline](/security/benchmark/azure/baselines/key-vault-security-baseline)
- [Secure your Azure Managed HSM deployment](../managed-hsm/secure-managed-hsm.md)
- [Virtual network service endpoints for Azure Key Vault](overview-vnet-service-endpoints.md)
- [Azure RBAC: Built-in roles](/azure/role-based-access-control/built-in-roles)
