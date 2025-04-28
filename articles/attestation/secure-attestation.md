---
title: Secure Azure Attestation
description: Learn how to secure Azure Attestation, with best practices for network security, identity management, and access control.
author: msmbaldwin
ms.author: mbaldwin
ms.service: security
ms.topic: conceptual
ms.custom: horz-security

ms.date: 03/28/2025
---

# Secure Microsoft Azure Attestation

Microsoft Azure Attestation is a service that enables you to verify the trustworthiness of platform components and applications running in Trusted Execution Environments (TEEs). It helps establish a root of trust by validating that TEEs are genuine, unmodified, and running the expected software. When using Azure Attestation for security-critical verification workflows, you must take steps to maximize the security of your attestation providers and the data processed through them.

This article provides guidance on how to best secure your Azure Attestation deployment.

## Network security

Azure Attestation supports network security features to help protect your attestation providers and their data from unauthorized access.

- **Use Azure Private Link for secure access**: Configure Azure Private Link to securely connect to Azure Attestation from your virtual network, preventing exposure of attestation endpoints to the public internet. Private Link provides a private connection that uses private IP addresses, effectively bringing Azure Attestation into your virtual network. For more details, see [Create private endpoint for Azure Attestation using PowerShell](private-endpoint-powershell.md).

- **Disable public network access when unnecessary**: By default, Azure Attestation has public network access disabled to enhance security. Keep this setting unless your architecture specifically requires public endpoint access. This minimizes the attack surface and exposure of your attestation providers. For more information about network security, see [Azure Private Link](/azure/private-link/private-link-overview).

## Identity management

Proper identity management is crucial for securing access to your attestation providers and their policies.

- **Use Microsoft Entra ID authentication**: Azure Attestation supports Microsoft Entra ID authentication for data plane access by default. Use this built-in integration to apply your organization's identity governance controls to attestation operations. This provides centralized authentication, detailed audit logs, and the ability to enforce conditional access policies. For more information, see [Role and permission requirements for policy management in Azure Attestation](quickstart-powershell.md#policy-management).

- **Leverage managed identities for applications**: When applications need to interact with Azure Attestation, use Azure managed identities to authenticate without storing credentials in your code. This eliminates the risk associated with hardcoded secrets and simplifies credential management. Managed identities are supported for data plane operations in Azure Attestation. For more information, see [What are managed identities for Azure resources?](/entra/identity/managed-identities-azure-resources/overview).

- **Implement service principals for automated processes**: For automation scenarios where managed identities aren't applicable, create dedicated service principals with least-privilege permissions to authenticate to Azure Attestation. Regularly rotate credentials for these service principals and monitor their usage patterns. For more information, see [Create an Azure service principal](/powershell/azure/create-azure-service-principal-azureps).

## Privileged access

Controlling privileged access is essential to prevent unauthorized modifications to attestation policies and configurations.

- **Apply role-based access control**: Use Azure RBAC to manage access to Azure Attestation data plane actions. Assign the Attestation Contributor role to users who need to perform operations like policy configuration or policy signer certificates management. For more details on role and permission requirements, see [Role and permission requirements for policy management in Azure Attestation](quickstart-powershell.md#policy-management).

## Logging and threat detection

Monitoring and logging are key components of a comprehensive security strategy for Azure Attestation.

- **Enable diagnostic logging**: Configure Azure resource logs to capture detailed information about attestation operations. These logs can help with security investigations, compliance requirements, and operational troubleshooting. For more details, see [Azure Attestation logging](view-logs.md).

- **Configure alerts for suspicious activities**: Set up alerts in Azure Monitor to notify you of unusual patterns or potential security incidents involving your attestation providers. Focus on high-risk operations like policy changes or unexpected authentication failures. For more information about setting up alerts, see [Azure Monitor alerts](/azure/azure-monitor/alerts/alerts-overview).

## Data protection

Protecting data processed by Azure Attestation is critical for maintaining the integrity of your attestation workflows.

- **Leverage automatic data encryption**: Azure Attestation automatically encrypts all data at rest using Microsoft-managed keys. This encryption helps protect attestation policies, certificates, and other stored data from unauthorized access. For more information, see [Data at Rest Encryption in Azure Attestation](basic-concepts.md#encryption-of-data-at-rest).

- **Use secure transport for all communications**: All data in transit to and from Azure Attestation is automatically encrypted using TLS 1.2 or higher. Enforce the use of secure protocols in your applications that communicate with Azure Attestation endpoints. For more information about data encryption in transit, see [Azure encryption overview](/azure/security/fundamentals/encryption-overview#encryption-of-data-in-transit).

## Backup and recovery

While Azure Attestation handles the underlying infrastructure, you should plan for backup and recovery of your custom attestation policies and configurations.

- **Document attestation policy configurations**: Maintain documentation of your attestation policy configurations to enable quick recovery in case of accidental deletion or corruption. Include details of policy signers, policy contents, and any custom settings.

- **Implement policy backup procedures**: Regularly export your attestation policies to a secure location. This ensures you can quickly restore them if needed without disrupting your attestation workflows.

- **Test recovery procedures regularly**: Periodically test your ability to restore attestation policies and configurations to ensure your recovery procedures work as expected and meet your business continuity requirements.

## Next steps

- [Learn more about Azure Attestation service](/azure/attestation/overview)
- [Set up an attestation provider using PowerShell](quickstart-powershell.md)
- [Create private endpoints for Azure Attestation](private-endpoint-powershell.md)
- [Explore Azure security best practices](/azure/security/fundamentals/best-practices-and-patterns)