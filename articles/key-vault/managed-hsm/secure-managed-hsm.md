---
title: Secure your Azure Key Vault Managed HSM deployment
description: Learn how to enhance the security of your Azure Key Vault Managed HSM deployment using best practices in identity, network, data protection, and access control.
author: msmbaldwin
ms.author: mbaldwin
ms.service: security
ms.topic: conceptual
ms.date: 03/25/2025
---

# Secure your Azure Key Vault Managed HSM deployment

Azure Key Vault Managed HSM is a cloud-based hardware security module that protects cryptographic keys and secrets. Because Managed HSM is often used to safeguard highly sensitive or business-critical data, it's vital to secure access to your HSMs, configure them according to security best practices, and monitor them for threats.

This article provides guidance on how to best secure your Azure Key Vault Managed HSM deployment.

## Network security

The Network Security section provides guidance on protecting Managed HSM with secure network connectivity, private endpoints, and network access controls. It helps reduce the exposure of your HSM to public networks and ensures that only authorized traffic is allowed.

- **Deploy private endpoints using Azure Private Link**: Use private endpoints to establish private, secured connectivity to your Managed HSM instance. This prevents exposure to the public internet and reduces attack vectors. For more details, see [Integrate Managed HSM with Azure Private Link](/azure/key-vault/managed-hsm/private-link).

- **Disable public network access**: Prevent access to the service from public IP addresses by disabling public network access in the Managed HSM configuration. For more details, see [Integrate Managed HSM with Azure Private Link](/azure/key-vault/managed-hsm/private-link).

## Identity management

This section focuses on securing authentication and identity access to your Managed HSM resources. Microsoft Entra provides a centralized identity solution for managing access to the HSM's management and data planes.

- **Require Microsoft Entra authentication for data plane access**: Microsoft Entra is used by default for authenticating data plane operations in Managed HSM, enabling centralized and secure identity control. For more details, see [Managed HSM access control](/azure/key-vault/managed-hsm/access-control).

- **Use managed identities for secure application access**: Managed identities are enabled by default, allowing applications to authenticate to Managed HSM without storing credentials. For more details, see [Secure access to your managed HSMs](/azure/key-vault/managed-hsm/secure-your-managed-hsm).

- **Authenticate using service principals when appropriate**: Use service principals for automated scenarios and workload-based access. For more details, see [Managed HSM access control](/azure/key-vault/managed-hsm/access-control).

- **Use conditional access policies to control access**: Define Conditional Access policies in Microsoft Entra to restrict access based on conditions like user risk, location, or device compliance. For more details, see [Conditional Access](/entra/identity/conditional-access/overview).

- **Use security groups for administrative access**: Assign the HSM Administrator role to a [Microsoft Entra security group](/entra/fundamentals/concept-learn-about-groups) instead of individual users. This reduces the risk of accidental lockout if a user account is deleted. For guidance, see [Managed HSM access control](/azure/key-vault/managed-hsm/access-control).

## Privileged access

The Privileged Access section emphasizes securing administrative actions and enforcing least-privilege access principles to reduce the risk of unauthorized or excessive permissions.

- **Lock down access to subscriptions and resource groups**: Use Azure RBAC to control administrative access at the management group, subscription, and resource group levels. For guidance, see [Azure RBAC overview](/azure/role-based-access-control/overview).
- 
- **Use least-privilege access principles when assigning roles**: Grant only the minimum set of permissions required. Regularly review role assignments. For more details, see [Managed HSM role management](/azure/key-vault/managed-hsm/role-management).

- **Avoid assigning multiple roles to the same identity**: Maintain separation of duties by ensuring a single user or identity isn't assigned conflicting roles. For more details, see [Access control](/azure/key-vault/managed-hsm/access-control).

- **Create custom roles with precise permissions**: Define custom roles to meet specific access requirements while maintaining a secure permission set. For more details, see [Access control](/azure/key-vault/managed-hsm/access-control).

- **Create per-key role assignments using local RBAC**: Use Managed HSM’s local RBAC model to control access at the individual key level. For more details, see [Managed HSM local RBAC](/azure/key-vault/managed-hsm/access-control#data-plane-and-managed-hsm-local-rbac).

## Logging and threat detection

This section covers how to enable audit logging and security monitoring for your HSM deployment, which is critical for detecting threats and conducting forensic investigations.

- **Enable resource logging for auditing and monitoring**: Turn on logging to track access and operations on your HSM. Configure the logs to send data to Log Analytics or a storage account. For more details, see [Managed HSM logging](/azure/key-vault/managed-hsm/logging).

- **Set up alerts for security events**: Configure alerts based on log data to receive notifications about unusual activity. For more details, see [Managed HSM logging](/azure/key-vault/managed-hsm/logging).

## Backup and recovery

The Backup and Recovery section helps you protect against accidental or malicious data loss by ensuring proper backup and retention policies.

- **Create regular HSM and key-level backups**: Schedule backups of the HSM and individual keys to prevent data loss. For more details, see [Full backup and restore](/azure/key-vault/managed-hsm/backup-restore).

- **Turn on purge protection**: Enable purge protection to prevent permanent deletion of the HSM or its keys before the retention period expires. For more details, see [Soft-delete overview](/azure/key-vault/managed-hsm/soft-delete-overview).

- **Retain soft-deleted resources**: Soft-delete is enabled by default. Choose a retention period of 7 to 90 days during which deleted items can be recovered. For more details, see [Soft-delete overview](/azure/key-vault/managed-hsm/soft-delete-overview).

## Posture and vulnerability management

The posture and vulnerability management section provides guidance on how to use policy enforcement tools to maintain security compliance across your environment.

- **Use Azure Policy to enforce configuration compliance**: Define and apply Azure Policy rules to audit or block insecure configurations. Integrate with Microsoft Defender for Cloud and Azure Monitor for visibility. For more details, see [Create and manage policies](/azure/governance/policy/tutorials/create-and-manage).

## Next steps

- See the full [Azure Managed HSM security baseline](/security/benchmark/azure/baselines/key-vault-managed-hsm-security-baseline).
- [Integrate with Azure Private Link](private-link.md)
- [Access control](access-control.md)
- [Local RBAC built-in roles](built-in-roles.md)
- [Integrate with Azure Policy](azure-policy.md)
- [Security domain overview](security-domain.md)
- [Azure Security fundamentals](/azure/security/fundamentals)
