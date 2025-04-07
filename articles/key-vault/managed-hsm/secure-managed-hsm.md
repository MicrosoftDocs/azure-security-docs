---
title: Secure your Azure Key Vault Managed HSM deployment
description: Learn how to enhance the security of your Azure Key Vault Managed HSM deployment using best practices in identity, network, data protection, and access control.
author: msmbaldwin
ms.author: mbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/25/2025
ms.custom: horz-security
ai-usage: ai-assisted

# CustomerIntent: As a security administrator, I want to implement best practices for securing my Managed HSM deployment so I can protect my organization's sensitive cryptographic keys, maintain regulatory compliance, and prevent unauthorized access to critical secrets.

---

# Secure your Azure Managed HSM deployment

Azure Key Vault Managed HSM is a cloud-based hardware security module that protects cryptographic keys. Because Managed HSM is often used to safeguard highly sensitive or business-critical data, it's vital to secure access to your HSMs, configure them according to security best practices, and monitor them for threats.

This article provides guidance on how to best secure your Azure Key Vault Managed HSM deployment.

## Network security

Network security covers protecting Managed HSM with secure network connectivity, private endpoints, and network access controls. It helps reduce the exposure of your HSM to public networks and ensures that only authorized traffic is allowed.

- **Deploy private endpoints using Azure Private Link**: Use private endpoints to establish private, secured connectivity to your Managed HSM instance. This prevents exposure to the public internet and reduces attack vectors. For more details, see [Integrate Managed HSM with Azure Private Link](private-link.md).

- **Disable public network access**: Prevent access to the service from public IP addresses by disabling public network access in the Managed HSM configuration. For more details, see [Integrate Managed HSM with Azure Private Link](private-link.md).

## Identity management

Identity management focuses on securing authentication and identity access to your Managed HSM resources. Microsoft Entra provides a centralized identity solution for managing access to the HSM's management and data planes.

- **Require Microsoft Entra authentication for data plane access**: Microsoft Entra is used by default for authenticating data plane operations in Managed HSM, enabling centralized and secure identity control. For more details, see [Microsoft Entra authentication](/entra/identity/authentication/overview-authentication).

- **Use managed identities for secure application access**: Managed identities are enabled by default, allowing applications to authenticate to Managed HSM without storing credentials. For more details, see [Microsoft Entra managed identities](/entra/identity/managed-identities-azure-resources/overview).

- **Authenticate using service principals when appropriate**: Use service principals for automated scenarios and workload-based access. For more details, see [Managed HSM role management](role-management.md).

- **Use conditional access policies to control access**: Define Conditional Access policies in Microsoft Entra to restrict access based on conditions like user risk, location, or device compliance. For more details, see [Conditional Access](/entra/identity/conditional-access/overview).

- **Use security groups for administrative access**: Assign the HSM Administrator role to a [Microsoft Entra security group](/entra/fundamentals/concept-learn-about-groups) instead of individual users. This reduces the risk of accidental lockout if a user account is deleted. For guidance, see [Access control for Managed HSM](access-control.md).

## Privileged access

Privileged access emphasizes securing administrative actions and enforcing least-privilege access principles to reduce the risk of unauthorized or excessive permissions.

- **Lock down access to subscriptions and resource groups**: Use Azure RBAC to control administrative access at the management group, subscription, and resource group levels. For guidance, see [Azure RBAC overview](/azure/role-based-access-control/overview).

- **Use least-privilege access principles when assigning roles**: Grant only the minimum set of permissions required. Regularly review role assignments. For more details, see [Managed HSM role management](role-management.md).

- **Avoid assigning multiple roles to the same identity**: Maintain separation of duties by ensuring a single user or identity isn't assigned conflicting roles. For more details, see [Access control for Managed HSM](access-control.md).

- **Create custom roles with precise permissions**: Define custom roles to meet specific access requirements while maintaining a secure permission set. For more details, see [Access control for Managed HSM](access-control.md).

- **Create per-key role assignments using local RBAC**: Use Managed HSM's local RBAC model to control access at the individual key level. For more details, see [Managed HSM local RBAC](access-control.md#data-plane-and-managed-hsm-local-rbac).

- **Enable Privileged Identity Management (PIM) for administrative roles**: Use [Microsoft Entra Privileged Identity Management](/entra/id-governance/privileged-identity-management/pim-configure) to enforce just-in-time access and reduce the risk of standing administrative privileges. For more details, see [Access control for Managed HSM: Privileged Identity Management](access-control.md#microsoft-entra-privileged-identity-management-pim).

## Logging and threat detection

Logging and threat detection help you monitor access and operations on your Managed HSM, enabling you to detect suspicious activity and ensure compliance with security policies.

- **Enable audit logging**: Configure diagnostic settings to enable audit logging for your Managed HSM. Logs capture all authenticated REST API requests, key operations, and security domain actions. Logs can be sent to an Azure storage account, Log Analytics workspace, or Event Hubs. For more details, see [Managed HSM logging](logging.md).

- **Set up alerts for critical events**: Configure Azure Monitor alerts to notify you of critical events, such as failed access attempts or unusual activity. Alerts can be based on metrics or log queries. For more details, see [Configure Managed HSM alerts](logging-azure-monitor.md#alerts).
- 
- **Analyze logs with Azure Monitor**: Use Azure Monitor to collect and analyze logs from your Managed HSM. Logs can be queried using Log Analytics and visualized in dashboards or workbooks. For more details, see [Monitor Azure Managed HSM](logging-azure-monitor.md).

- **Retain logs for compliance and investigations**: Ensure that logs are retained for an appropriate duration to meet compliance requirements and support forensic investigations. Use Azure Monitor Log Analytics retention policies to manage log storage. For more details, see [Log retention in Azure Monitor](logging-azure-monitor.md#data-storage).

## Backup and recovery

Backup and recovery helps you protect against accidental or malicious data loss by ensuring proper backup and retention policies.

- **Create regular HSM and key-level backups**: Schedule backups of the HSM and individual keys to prevent data loss. For more details, see [Full backup and restore](backup-restore.md).

- **Turn on purge protection**: Enable purge protection to prevent permanent deletion of the HSM or its keys before the retention period expires. For more details, see [Soft-delete overview](soft-delete-overview.md).

- **Retain soft-deleted resources**: Soft-delete is enabled by default. Choose a retention period of 7 to 90 days during which deleted items can be recovered. For more details, see [Soft-delete overview](soft-delete-overview.md).

- **Protect the security domain to prevent cryptographic lockout**: Follow best practices for managing the security domain and its keys to ensure business continuity and prevent unauthorized access. For more details, see [Security domain overview](security-domain.md).

## Posture and vulnerability management

Posture and vulnerability management provides guidance on how to use policy enforcement tools to maintain security compliance across your environment.

- **Use Azure Policy to enforce configuration compliance**: Define and apply [Azure Policy](/azure/governance/policy/overview) rules to audit or block insecure configurations. Integrate with [Microsoft Defender for Cloud](/azure/defender-for-cloud/defender-for-cloud-introduction) and [Azure Monitor](/azure/azure-monitor/fundamentals/overview) for visibility. For more details, see [Create and manage policies](/azure/governance/policy/tutorials/create-and-manage).



## Next steps

- [Integrate with Azure Private Link](private-link.md)
- [Access control](access-control.md)
- [Local RBAC built-in roles](built-in-roles.md)
- [Integrate with Azure Policy](azure-policy.md)
- [Security domain overview](security-domain.md)
- [Azure Security fundamentals](/azure/security/fundamentals)
