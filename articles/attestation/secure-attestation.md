---
title: Secure your Azure Attestation deployment
description: Learn how to secure Azure Attestation, with best practices for network security, identity management, and access control.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-attestation
ms.topic: best-practice
ms.custom: horz-security
ms.date: 07/10/2026
ai-usage: ai-assisted
---

# Secure your Azure Attestation deployment

Microsoft Azure Attestation is a service that helps you verify the trustworthiness of platform components and applications running in Trusted Execution Environments (TEEs). Azure Attestation helps establish a root of trust by validating that TEEs are genuine, unmodified, and running the expected software. When you use Azure Attestation for security-critical verification workflows, take steps to maximize the security of your attestation providers and the data processed through them.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

This article provides security recommendations to help protect your Azure Attestation deployment.

## Network security

Azure Attestation supports network security features to help protect your attestation providers and their data from unauthorized access.

- **Use Azure Private Link for secure access**: Configure Azure Private Link to securely connect to Azure Attestation from your virtual network, preventing exposure of attestation endpoints to the public internet. Private Link provides a private connection that uses private IP addresses, effectively bringing Azure Attestation into your virtual network. For more information, see [Create private endpoint for Azure Attestation using PowerShell](private-endpoint-powershell.md).

- **Disable public network access when unnecessary**: Azure Attestation supports the `publicNetworkAccess` property on attestation providers. Set it to `Disabled` for providers reached exclusively through Private Link so no data-plane traffic can arrive from the public internet. For more information, see [Azure Private Link](/azure/private-link/private-link-overview).

## Identity and access management

Proper identity and access management is crucial for securing access to your attestation providers, their policies, and their signing certificates. Azure Attestation uses Microsoft Entra ID for data-plane authentication and Azure RBAC for authorization.

- **Use Microsoft Entra ID authentication**: Azure Attestation authenticates data-plane callers through Microsoft Entra ID by default. Use this built-in integration to apply your organization's identity governance controls (Conditional Access, MFA, session policies) to attestation operations. For more information, see [Role and permission requirements for policy management in Azure Attestation](quickstart-powershell.md#policy-management).

- **Use managed identities for applications**: When applications need to call Azure Attestation, authenticate them by using Azure managed identities so you don't store secrets or certificates in code. Azure Attestation supports managed identities for data-plane operations. For more information, see [What are managed identities for Azure resources?](/entra/identity/managed-identities-azure-resources/overview).

- **Use service principals only when managed identities can't be used**: For automation scenarios where managed identities don't apply (for example, cross-tenant scenarios), create dedicated service principals with least-privilege permissions. Rotate credentials regularly and monitor their sign-in activity. For more information, see [Create an Azure service principal](/powershell/azure/create-azure-service-principal-azureps).

- **Apply Azure RBAC with the Attestation Contributor role**: Assign the built-in **Attestation Contributor** role only to users and service principals that need to manage policies or policy-signing certificates on an attestation provider. Grant read-only access with the **Attestation Reader** role. For more information, see [Role and permission requirements for policy management in Azure Attestation](quickstart-powershell.md#policy-management).

- **Enforce PIM for privileged assignments**: Use Microsoft Entra Privileged Identity Management (PIM) to make the **Attestation Contributor** role eligible rather than permanently assigned, and require MFA plus approval on activation. For more information, see [Privileged Identity Management overview](/entra/id-governance/privileged-identity-management/pim-configure).

## Data protection

Azure Attestation processes attestation evidence and policies whose integrity is central to establishing trust in your TEEs. Data-at-rest and data-in-transit encryption are enabled by default, and the actionable controls are around policy signing.

- **Sign your custom attestation policies**: Use signed policies (rather than unsigned policies) when your workload requires strong provenance for policy changes. Signed policies let you verify that a policy came from a trusted policy signer and wasn't tampered with in transit. For more information, see [How to author and sign an attestation policy](author-sign-policy.md).

- **Protect policy-signing certificate private keys**: Store the private keys used to sign attestation policies in Azure Key Vault (or an HSM you control) and grant access only to the identities that must sign policies. Rotate these keys on a schedule aligned with your organization's cryptographic policy. For more information, see [About Azure Key Vault](/azure/key-vault/general/overview).

- **Understand default encryption behavior**: Azure Attestation automatically encrypts all data at rest by using Microsoft-managed keys, and all data in transit uses TLS 1.2 or higher. You don't need to configure anything for the default posture. Validate that your client applications negotiate TLS 1.2 or higher against the attestation endpoint. For more information, see [Encryption of data at rest](basic-concepts.md#encryption-of-data-at-rest).

## Logging and monitoring

Monitoring and logging are key components of a comprehensive security strategy for Azure Attestation.

- **Enable diagnostic logging**: Configure Azure resource logs to capture detailed information about attestation operations. These logs support security investigations, compliance requirements, and operational troubleshooting. For more information, see [Azure Attestation logging](view-logs.md).

- **Configure alerts for suspicious activities**: Set up Azure Monitor alerts to notify you of unusual patterns or potential security incidents involving your attestation providers. Focus on high-risk operations like policy changes, policy-signer certificate rotations, and unexpected authentication failures. For more information, see [Azure Monitor alerts](/azure/azure-monitor/alerts/alerts-overview).

## Compliance and governance

Governance controls help you enforce a consistent security posture across attestation providers as your estate grows.

- **Enforce configuration with Azure Policy**: Use Azure Policy definitions to audit or enforce required configurations on attestation providers - for example, requiring `publicNetworkAccess: Disabled` or restricting the regions where providers can be created. For more information, see [Azure Policy built-in definitions for Azure Attestation](policy-reference.md).

- **Adopt the Microsoft cloud security benchmark**: Align your attestation posture with the [Microsoft cloud security benchmark (MCSB)](/security/benchmark/azure/overview), which provides horizontal controls for network security, identity, data protection, and monitoring that also apply to Azure Attestation.

## Backup and recovery

Azure Attestation manages the underlying infrastructure. Your recovery responsibilities are limited to your custom attestation policies and policy-signing certificates.

- **Version-control your policy artifacts**: Store your attestation policy source files (and the source used to build signed policies) in a version-controlled repository so you can recreate any deployed policy deterministically. Combined with signed policies, this approach provides an auditable recovery path.

- **Back up policy-signing certificates**: If you use signed policies, store the policy-signing certificate in an HSM-backed store such as Azure Key Vault and back it up according to your organization's key management procedures. If you lose the signing key, you can't create new signed policy versions under the same signer identity.

- **Test recovery periodically**: Regularly exercise the flow of redeploying your attestation policy to a test attestation provider so a validated procedure is ready if a production policy is lost or corrupted.

## Next steps

- [Overview of Azure Attestation](/azure/attestation/overview)
- [Set up an attestation provider using PowerShell](quickstart-powershell.md)
- [Create private endpoints for Azure Attestation](private-endpoint-powershell.md)
- [Azure security best practices](/azure/security/fundamentals/best-practices-and-patterns)