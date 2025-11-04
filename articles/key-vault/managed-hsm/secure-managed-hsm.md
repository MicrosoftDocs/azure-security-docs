---
title: Secure your Azure Managed HSM deployment
description: Learn how to enhance the security of your Azure Managed HSM deployment using best practices in identity, network, data protection, and access control.
author: msmbaldwin
ms.author: mbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 10/02/2025
ms.custom: horz-security
ai-usage: ai-assisted

# CustomerIntent: As a security administrator, I want to implement best practices for securing my Managed HSM deployment so I can protect my organization's sensitive cryptographic keys, maintain regulatory compliance, and prevent unauthorized access to critical secrets.

---

# Secure your Azure Managed HSM deployment

Azure Managed HSM is a fully managed, highly available, single-tenant Hardware Security Module (HSM) service that provides FIPS 140-3 Level 3 validated cryptographic key protection for your cloud applications. Because Managed HSM safeguards your most sensitive cryptographic keys and secrets, implementing comprehensive security controls is essential to protect against threats and maintain business continuity.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

This article provides security recommendations to help protect your Azure Managed HSM deployment.

## Network security

Network security protects your Managed HSM through secure connectivity and network access controls. These network security features are listed from most restricted to least restricted capabilities. Pick the configuration that best suits your organization's use case. For detailed information about all network security configurations, see [Network security for Azure Key Vault Managed HSM](network-security.md).

- **Disable public network access and use Private Endpoints only**: Deploy Azure Private Link to establish private, secured connectivity to your Managed HSM instance by creating a private endpoint in your virtual network. Disabling public network access prevents access from public IP addresses by configuring your Managed HSM to deny public network access. This prevents exposure to the public internet and routes all traffic over the Microsoft backbone network. See [Integrate Managed HSM with Azure Private Link](private-link.md).

- **Configure Managed HSM firewall with trusted services**: Configure Managed HSM firewall rules to deny public internet access while allowing specific trusted Azure services through the `--bypass AzureServices` setting when required by your scenario. This restricts the attack surface while maintaining necessary service integrations. For full details, see [Network security: Azure Managed HSM Firewall Enabled (Trusted Services Only)](network-security.md#managed-hsm-firewall-enabled-trusted-services).

- **Enable IP Firewall (PREVIEW)**: Limit access to public static IP addresses when network scenarios require controlled public access. For full details, see [Azure Key Vault Managed HSM network security: firewall settings](network-security.md#firewall-settings).

For step-by-step configuration instructions, see [How to configure Azure Managed HSM networking settings](configure-network-security.md).

## Identity and access management

Identity and access management secures authentication and authorization to your Managed HSM resources. Managed HSM uses a dual-plane access model with different authorization systems for control plane and data plane operations.

- **Implement Managed HSM local RBAC for data plane access**: Use Managed HSM local RBAC to control access to keys and cryptographic operations within the HSM. This authorization system operates independently from Azure RBAC and provides granular permissions for key operations, role assignments, and security domain management. See [Access control for Managed HSM](/azure/key-vault/managed-hsm/access-control).

- **Enable managed identities for application access**: Configure system-assigned or user-assigned managed identities for applications to authenticate to Managed HSM without storing credentials in code or configuration files. Managed identities integrate with Microsoft Entra ID and automatically handle credential rotation. See [Access control for Managed HSM](/azure/key-vault/managed-hsm/access-control).

- **Apply least-privilege access with appropriate scopes**: Grant permissions at the most restrictive scope necessary—either HSM-level (`/` or `/keys`) for broad access or key-level (`/keys/<key-name>`) for specific key access. Use built-in roles like Managed HSM Crypto Officer, Managed HSM Crypto User, or Managed HSM Crypto Auditor based on required operations. See [Access control for Managed HSM](/azure/key-vault/managed-hsm/access-control).

- **Assign HSM Administrator role to security groups**: Grant the HSM Administrator role to Microsoft Entra security groups instead of individual users to prevent accidental lockout if user accounts are deleted. This approach simplifies permission management and ensures continuity of administrative access during the HSM provisioning process. See [Access control for Managed HSM](/azure/key-vault/managed-hsm/access-control).

- **Enable Privileged Identity Management for administrative roles**: Use Microsoft Entra Privileged Identity Management (PIM) to enforce just-in-time access for highly privileged roles like Managed HSM Administrator. PIM reduces the risk of standing administrative privileges and provides approval workflows for elevated access. See [Access control for Managed HSM](/azure/key-vault/managed-hsm/access-control).

- **Separate control plane and data plane access**: Understand that control plane access (Azure RBAC) for managing HSM resources does not grant data plane access to keys. Explicitly assign data plane roles through Managed HSM local RBAC to users who need to perform key operations. See [Access control for Managed HSM](/azure/key-vault/managed-hsm/access-control).

## Data protection

Data protection safeguards cryptographic keys and sensitive data stored in Managed HSM through encryption, key management policies, and secure storage practices. Proper data protection ensures key material remains confidential and tamper-resistant.

- **Implement multi-person control for security domain**: Configure a security domain quorum with multiple RSA key pairs (minimum 3 recommended) to prevent single-person control over HSM recovery. Specify a quorum threshold that requires multiple key holders to collaborate for security domain decryption, ensuring no single individual can compromise the HSM. See [Security domain overview](/azure/key-vault/managed-hsm/security-domain).

- **Store security domain keys offline in secure locations**: Keep security domain private keys on encrypted, offline storage devices such as encrypted USB drives stored in separate geographical locations within physical safes or lock boxes. Never store security domain keys on internet-connected computers to reduce exposure to cyber threats and ensure air-gapped security. See [Security domain overview](/azure/key-vault/managed-hsm/security-domain).

- **Establish security domain key management procedures**: Implement policies for periodic review of security domain key custody when personnel changes occur or when keys may be compromised. Document security domain holder responsibilities, maintain accurate records of key locations and custody, and ensure the quorum can be assembled for disaster recovery scenarios. See [Security domain overview](/azure/key-vault/managed-hsm/security-domain).

- **Enable purge protection for HSM and keys**: Configure purge protection to prevent permanent deletion of the HSM or individual keys before the retention period expires. This control protects against accidental or malicious deletion and provides a recovery window for critical operations. See [Soft-delete overview](/azure/key-vault/managed-hsm/soft-delete-overview).

- **Configure appropriate soft-delete retention periods**: Set soft-delete retention periods between 7 to 90 days based on your recovery requirements and compliance needs. Longer retention periods provide more recovery time but may conflict with data residency requirements. See [Soft-delete overview](/azure/key-vault/managed-hsm/soft-delete-overview).

## Logging and monitoring

Logging and monitoring provide visibility into HSM access patterns and operations, enabling threat detection and compliance reporting. Comprehensive logging helps identify suspicious activity and supports forensic investigations.

- **Enable audit logging with diagnostic settings**: Configure diagnostic settings to capture all authenticated REST API requests, key operations, and security domain actions in the AzureDiagnostics table. Route logs to Azure Storage accounts, Log Analytics workspaces, or Event Hubs based on your retention and analysis requirements. See [Managed HSM logging](/azure/key-vault/managed-hsm/logging).

- **Analyze logs with Azure Monitor and Log Analytics**: Use Azure Monitor to collect and analyze HSM logs through KQL queries that filter on ResourceProvider "MICROSOFT.KEYVAULT" and ResourceType "MANAGEDHSMS". Create custom dashboards and workbooks for security operations teams to monitor access patterns and key usage. See [Monitor Azure Managed HSM](/azure/key-vault/managed-hsm/logging-azure-monitor).

- **Configure alerts for critical security events**: Create Azure Monitor alert rules for events such as HSM availability drops below 100%, service API latency exceeding thresholds, unusual error code patterns, or failed authentication attempts. Configure both static threshold and dynamic threshold alerts to reduce false positives while maintaining security visibility. See [Configure Managed HSM alerts](/azure/key-vault/managed-hsm/configure-alerts).

- **Integrate with Microsoft Sentinel for advanced threat detection**: Deploy Microsoft Sentinel to automatically detect suspicious activity using machine learning analytics and custom detection rules specific to Managed HSM operations. Create analytic rules for sensitive operations like security domain downloads, bulk key operations, or anomalous access patterns. See [Setting up Microsoft Sentinel for Azure Managed HSM](/azure/key-vault/managed-hsm/sentinel).

- **Implement proper log retention policies**: Establish log retention periods that meet compliance requirements and support forensic investigations. Use Azure Monitor Log Analytics retention policies to manage storage costs while maintaining adequate investigation capabilities for security incidents. See [Monitor Azure Managed HSM](/azure/key-vault/managed-hsm/logging-azure-monitor).

## Compliance and governance

Compliance and governance controls ensure your Managed HSM deployment meets regulatory requirements and organizational policies through automated policy enforcement and compliance monitoring.

- **Implement Azure Policy for key governance**: Grant the "Managed HSM Crypto Auditor" role to the "Azure Managed HSM Key Governance Service" (App ID: a1b76039-a76c-499f-a2dd-846b4cc32627) to enable Azure Policy compliance scanning, then define policy rules to audit or enforce secure key configurations including key expiration requirements, minimum RSA key sizes, and elliptic curve algorithm restrictions. Without this role assignment, Azure Policy cannot evaluate your complete key inventory. See [Integrate Azure Managed HSM with Azure Policy](/azure/key-vault/managed-hsm/azure-policy).

- **Configure key lifecycle and cryptographic standards**: Use built-in Azure Policy definitions to enforce key expiration dates, mandate minimum RSA key sizes for security compliance, restrict elliptic curve cryptography to approved curve names (P-256, P-256K, P-384, P-521), and ensure keys have sufficient time before expiration for rotation procedures. See [Integrate Azure Managed HSM with Azure Policy](/azure/key-vault/managed-hsm/azure-policy).

- **Monitor compliance through Azure Policy dashboards**: Use Azure Policy compliance dashboards to track adherence to cryptographic security standards and identify non-compliant keys that require remediation. Set up both audit and deny policy effects to provide visibility and enforcement of security baselines. See [Integrate Azure Managed HSM with Azure Policy](/azure/key-vault/managed-hsm/azure-policy).

## Backup and recovery

Backup and recovery protects against data loss and enables business continuity through proper backup strategies, disaster recovery procedures, and recovery testing to ensure cryptographic keys remain accessible.

- **Create regular full HSM backups**: Schedule automated full HSM backups that include all keys, versions, attributes, tags, and role assignments to prevent data loss from hardware failures or operational incidents. Use user-assigned managed identities for backup operations to enable secure access to storage accounts without credential management. See [Full backup and restore](/azure/key-vault/managed-hsm/backup-restore).

- **Implement individual key-level backups for critical keys**: Create selective backups of high-value keys using the `az keyvault key backup` command to enable granular recovery without full HSM restore operations. Key backups are encrypted and cryptographically tied to the security domain, meaning they can only be restored to HSMs sharing the same security domain. See [Full backup and restore](/azure/key-vault/managed-hsm/backup-restore).

- **Prepare comprehensive disaster recovery procedures**: Develop and test disaster recovery plans that include security domain recovery using RSA key pairs, backup restoration procedures, and application reconfiguration steps. Ensure you maintain secure offline access to security domain files, private keys (minimum quorum), and recent backups stored in geographically separate locations. See [Disaster recovery guide](/azure/key-vault/managed-hsm/disaster-recovery-guide).

- **Test backup and restore procedures regularly**: Conduct periodic disaster recovery drills using non-production HSM instances to validate security domain recovery, backup restoration, and the complete disaster recovery workflow. Testing ensures security domain quorum holders can successfully execute recovery operations and verifies backup integrity. See [Disaster recovery guide](/azure/key-vault/managed-hsm/disaster-recovery-guide).

- **Secure backup storage with proper access controls**: Store HSM backups in Azure Storage accounts configured with appropriate RBAC permissions, private endpoints, and customer-managed encryption keys. Configure the user-assigned managed identity with Storage Blob Data Contributor role and implement backup retention policies that balance recovery requirements with storage costs. See [Full backup and restore](/azure/key-vault/managed-hsm/backup-restore).

## Service-specific security

Service-specific security addresses unique characteristics of Managed HSM including hardware-level protection, FIPS compliance, and specialized cryptographic operations that distinguish it from other Azure services.

- **Leverage FIPS 140-3 Level 3 hardware validation**: Take advantage of Managed HSM's FIPS 140-3 Level 3 validated hardware security modules that provide tamper-resistant, high-entropy cryptographic processing with hardware-based key isolation. This provides the highest level of key protection available in Azure. See [What is Azure Managed HSM?](/azure/key-vault/managed-hsm/overview).

- **Implement bring-your-own-key (BYOK) for regulatory compliance**: Use BYOK to import HSM-protected keys from on-premises HSMs when regulatory requirements mandate specific key generation procedures. BYOK ensures keys never exist outside HSM boundaries in plaintext form during the transfer process. See [What is Azure Managed HSM?](/azure/key-vault/managed-hsm/overview).

- **Utilize single-tenant isolation for sensitive workloads**: Leverage Managed HSM's single-tenant architecture where each HSM instance is dedicated to a single customer and cryptographically isolated through customer-specific security domains. This provides stronger isolation than multitenant key vault solutions. See [What is Azure Managed HSM?](/azure/key-vault/managed-hsm/overview).

- **Implement proper key attestation procedures**: Use key attestation capabilities to prove that keys were generated and processed within FIPS 140-3 Level 3 hardware boundaries. Key attestation provides cryptographic proof of key provenance for high-assurance scenarios. See [Key attestation](/azure/key-vault/managed-hsm/key-attestation).

- **Configure cross-region replication for business continuity**: Enable multi-region replication to extend your Managed HSM from a primary region to an extended region, providing active-active deployment with automated replication. Both regions can serve requests, and the Traffic Manager routes requests to the closest available region, increasing SLA to 99.99% combined. See [Multi-region replication](/azure/key-vault/managed-hsm/multi-region-replication).

## Next steps

- [Network security for Azure Key Vault Managed HSM](network-security.md)
- [How to configure Azure Managed HSM networking settings](configure-network-security.md)
- [Integrate with Azure Private Link](private-link.md)
- [Access control](access-control.md)
- [Local RBAC built-in roles](built-in-roles.md)
- [Integrate with Azure Policy](azure-policy.md)
- [Security domain overview](security-domain.md)
- [Azure Security fundamentals](/azure/security/fundamentals)
