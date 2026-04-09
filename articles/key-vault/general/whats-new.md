---
title: What's new for Azure Key Vault
description: Recent updates for Azure Key Vault
services: key-vault
author: msmbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: reference
ms.date: 04/02/2026
ms.author: mbaldwin

#Customer intent: As an Azure Key Vault administrator, I want to react to soft-delete being turned on for all key vaults.

---

# What's new for Azure Key Vault

Here's what's new with Azure Key Vault. New features and improvements are also announced on the [Azure updates Key Vault channel](https://azure.microsoft.com/updates/?category=security&query=Key%20vault).

## March 2026

### New control plane API versions

New Azure Key Vault control plane API versions (2026-02-01 and 2026-03-01-preview) are now available across public Azure regions, with availability extended to Mooncake and Fairfax clouds.

### Preview API version deprecation

Azure Key Vault is deprecating older preview control plane API versions in accordance with Azure guidelines. All preview API versions except 2026-04-01-preview will be deprecated with a 90-day notice period. Customers using preview API versions should plan to migrate to the latest stable API version (2026-02-01) or the upcoming 2026-04-01-preview version.

### Supported control plane API versions

Use the latest stable version for production workloads. Preview versions are for early testing only and aren't covered by Azure support SLAs. Each link points to the API specification in the `azure-rest-api-specs` repository.

#### Stable API versions

- [2026-02-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2026-02-01) (recommended)
- [2025-05-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2025-05-01)
- [2024-11-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2024-11-01)
- [2023-07-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2023-07-01)
- [2023-02-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2023-02-01)
- [2022-11-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2022-11-01)
- [2022-07-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2022-07-01)
- [2021-10-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2021-10-01)
- [2019-09-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2019-09-01)
- [2018-02-14](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2018-02-14)
- [2016-10-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2016-10-01)
- [2015-06-01](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/stable/2015-06-01)

#### Preview API versions

- [2024-12-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2024-12-01-preview) (recommended)
- [2024-04-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2024-04-01-preview)
- [2022-02-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2022-02-01-preview)
- [2021-11-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2021-11-01-preview)
- [2021-06-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2021-06-01-preview)
- [2021-04-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2021-04-01-preview)
- [2020-04-01-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2020-04-01-preview)
- [2018-02-14-preview](https://github.com/Azure/azure-rest-api-specs/tree/main/specification/keyvault/resource-manager/Microsoft.KeyVault/KeyVault/preview/2018-02-14-preview)

### Control plane SDK releases

All five Azure Key Vault control plane management SDKs supporting API version 2026-02-01 are now released. Control plane management SDKs that use older API versions will stop working when those API versions retire on February 27, 2027. Upgrade to the following SDK versions or later:

| Language | Package | Version |
|----------|---------|---------|
| .NET | [Azure.ResourceManager.KeyVault](https://www.nuget.org/packages/Azure.ResourceManager.KeyVault) | 1.4.0 |
| JavaScript | [@azure/arm-keyvault](https://www.npmjs.com/package/@azure/arm-keyvault) | Latest |
| Go | [armkeyvault](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/keyvault/armkeyvault/v2) | v2 |
| Python | [azure-mgmt-keyvault](https://pypi.org/project/azure-mgmt-keyvault/) | Latest |
| Java | [azure-resourcemanager-keyvault](https://central.sonatype.com/artifact/com.azure.resourcemanager/azure-resourcemanager-keyvault) | Latest |

> [!NOTE]
> Data Plane SDKs are not affected by this change. Only control plane management SDKs need to be updated.

### Azure RBAC is the default access control model for new vaults

Azure Key Vault API version 2026-02-01 introduces Azure RBAC as the default access control model for newly created key vaults when using this API version. Existing vaults continue using their current access model unless explicitly updated. Both Azure RBAC and access policies remain fully supported.

For more information, see [Prepare for Key Vault API version 2026-02-01 and later](access-control-default.md).

### Private endpoint limit enforcement

Azure Key Vault is enforcing limits on the number of private endpoints per vault. Customers exceeding these limits may need to reduce usage or request an exception through Azure support.

For more information on private endpoint limits, see [Azure Key Vault service limits](service-limits.md).

## November 2025

### Security best practices articles

New security best practices documentation is available for Azure Key Vault workloads:

- [Secure your Azure Key Vault](secure-key-vault.md): Comprehensive security guidance for vault architecture, network security, identity and access management, and monitoring.
- [Secure your Azure Key Vault keys](../keys/secure-keys.md): Key-specific security recommendations for key types, protection levels, rotation, and operations.
- [Secure your Azure Key Vault secrets](../secrets/secure-secrets.md): Secrets-specific best practices for storage, rotation, and access patterns.
- [Secure your Azure Key Vault certificates](../certificates/secure-certificates.md): Certificate-specific guidance for lifecycle management, CA integration, and renewal.

### Apps, API keys, and Key Vault secrets guidance

New guidance is available for configuring applications to securely interact with API keys stored in Azure Key Vault, including best practices for access control, monitoring, and network restrictions.

For more information, see [Apps, API keys, and Azure Key Vault secrets](apps-api-keys-secrets.md).

## July 2023

A built-in policy now governs key rotation configuration in Azure Key Vault. With this policy, you can audit existing keys in key vaults to ensure that all keys are configured for rotation and comply with your organization's standards.

For more information, see [Configure key rotation governance](../keys/how-to-configure-key-rotation.md#configure-key-rotation-policy-governance).

## June 2023

Key Vault enforces TLS 1.2 or higher for enhanced security. If you're still using an older TLS version, see [Enable support for TLS 1.2 in your environment](/troubleshoot/entra/entra-id/ad-dmn-services/enable-support-tls-environment#why-this-change-is-being-made) to update your clients and ensure uninterrupted access to Key Vault services. To monitor the TLS version that clients use, see the [sample Kusto queries](monitor-key-vault.md#sample-kusto-queries) in the Key Vault monitoring guide.

## May 2023

Azure RBAC is now the recommended authorization system for the Azure Key Vault data plane. Azure RBAC is built on Azure Resource Manager and provides fine-grained access management of Azure resources. With Azure RBAC, you control access to resources by creating role assignments. Each role assignment consists of three elements: a security principal, a role definition (predefined set of permissions), and a scope (group of resources or individual resource).

For more information, see [Azure role-based access control (Azure RBAC) vs. access policies](rbac-access-policy.md).

## February 2023

A built-in policy to govern the migration to Azure role-based access control (Azure RBAC) is now in preview. With the built-in policy, you can audit existing key vaults and enforce all new key vaults to use the Azure RBAC permission model. To learn how to enforce the new built-in policy, see [Azure RBAC migration governance](../general/rbac-migration.md#migration-governance-with-azure-policy).

## April 2022

Automated encryption key rotation in Key Vault is now generally available.

For more information, see [Configure key auto-rotation in Key Vault](../keys/how-to-configure-key-rotation.md).

## January 2022

Azure Key Vault service throughput limits doubled for each vault to help ensure high performance for applications. For example, for secret GET and RSA 2,048-bit software keys, you now receive 4,000 GET transactions per 10 seconds instead of 2,000. The service quotas are specific to operation type. For the full list, see [Azure Key Vault service limits](./service-limits.md).

For the Azure update announcement, see [General availability: Azure Key Vault increased service limits for all its customers](https://azure.microsoft.com/updates/azurekeyvaultincreasedservicelimits/).


## December 2021

Automated encryption key rotation in Key Vault is now in preview. You can set a rotation policy on a key to schedule automated rotation and configure expiry notifications through Event Grid integration.

For more information, see [Configure key auto-rotation in Key Vault](../keys/how-to-configure-key-rotation.md).

## October 2021

Integration of Azure Key Vault with Azure Policy has reached general availability. Azure Policy lets you place guardrails on Key Vault and its objects to ensure they comply with your organization's security recommendations and compliance regulations. You can perform real-time policy-based enforcement and on-demand compliance assessment of existing secrets in your Azure environment. A compliance dashboard shows which resources and components are compliant.

Azure Policy for Key Vault provides a full suite of built-in policies for governance of your keys, secrets, and certificates. To learn more, see [Integrate Azure Key Vault with Azure Policy](./azure-policy.md?tabs=certificates). For the announcement, see [Azure Key Vault policy general availability](https://azure.microsoft.com/updates/gaazurepolicyforkeyvault).

## June 2021

Azure Key Vault Managed HSM is generally available. Managed HSM offers a fully managed, highly available, single-tenant cloud service to safeguard cryptographic keys for your cloud applications using FIPS 140-2 Level 3 validated HSMs.

For more information, see [Azure Key Vault Managed HSM overview](../managed-hsm/overview.md).

## February 2021

Azure role-based access control (RBAC) for Azure Key Vault data plane authorization is now generally available. You can now manage RBAC for Key Vault keys, certificates, and secrets with role assignment scopes from management group to individual key, certificate, and secret.

For more information, see [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](rbac-guide.md).

## October 2020

Two changes have been made to Azure Key Vault PowerShell cmdlets:

- The DisableSoftDelete and EnableSoftDelete parameters of [Update-AzKeyVault](/powershell/module/az.keyvault/update-azkeyvault) have been deprecated.
- The output of the [Get-AzKeyVaultSecret](/powershell/module/az.keyvault/get-azkeyvaultsecret) cmdlet no longer has the `SecretValueText` attribute.

## July 2020

> [!WARNING]
> This update has the potential to impact Azure Key Vault implementations.

### Azure TLS certificate changes

Microsoft is updating Azure services to use TLS certificates from a different set of Root Certificate Authorities (CAs). This change is being made because the current CA certificates don't comply with one of the CA/Browser Forum Baseline requirements.  For full details, see [Azure TLS Certificate Changes](/azure/security/fundamentals/tls-certificate-changes).

## June 2020

Azure Monitor for Key Vault is now in preview.  Azure Monitor provides comprehensive monitoring of your key vaults by delivering a unified view of your Key Vault requests, performance, failures, and latency. For more information, see [Azure Monitor for Key Vault (preview).](../key-vault-insights-overview.md).

## May 2020

Key Vault "bring your own key" (BYOK) is now generally available. See the [Azure Key Vault BYOK specification](../keys/byok-specification.md), and learn how to [Import HSM-protected keys to Key Vault (BYOK)](../keys/hsm-protected-keys-byok.md).

## March 2020

Private endpoints now available in preview. Azure Private Link Service enables you to access Azure Key Vault and Azure hosted customer/partner services over a Private Endpoint in your virtual network.  Learn how to [Integrate Key Vault with Azure Private Link](private-link-service.md).

## 2019

- Release of the next-generation Azure Key Vault SDKs. For examples of their use, see the Azure Key Vault secret quickstarts for [Python](../secrets/quick-create-python.md), [.NET](../secrets/quick-create-net.md), [Java](../secrets/quick-create-java.md), and [Node.js](../secrets/quick-create-node.md)
- New Azure policies to manage key vault certificates. See the [Azure Policy built-in definitions for Key Vault](../policy-reference.md).
- Azure Key Vault Virtual Machine extension now generally available.  See [Key Vault virtual machine extension for Linux](/azure/virtual-machines/extensions/key-vault-linux) and [Key Vault virtual machine extension for Windows](/azure/virtual-machines/extensions/key-vault-windows).
- Event-driven secrets management for Azure Key Vault now available in Azure Event Grid. For more information, see [the Event Grid schema for events in Azure Key Vault](/azure/event-grid/event-schema-key-vault), and learn how to [Receive and respond to key vault notifications with Azure Event Grid](event-grid-tutorial.md).

## 2018

New features and integrations released this year:

- Integration with Azure Functions. For an example scenario that uses [Azure Functions](/azure/azure-functions/) for key vault operations, see [Automate the rotation of a secret](../secrets/tutorial-rotation.md).
- [Integration with Azure Databricks](./integrate-databricks-blob-storage.md). With this, Azure Databricks now supports two types of secret scopes: Azure Key Vault-backed and Databricks-backed. For more information, see [Create an Azure Key Vault-backed secret scope](/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope)
- [Virtual network service endpoints for Azure Key Vault](overview-vnet-service-endpoints.md).

## 2016

New features released this year:

- Managed storage account keys (deprecated). Storage Account Keys feature added easier integration with Azure Storage. This feature is now deprecated; use [Microsoft Entra ID authorization for Azure Storage](/azure/storage/common/authorize-data-access) instead.
- Soft delete. Soft-delete feature improves data protection of your key vaults and key vault objects. For more information, see [Soft-delete overview](./soft-delete-overview.md).

## 2015

New features released this year:
- Certificate management. Added as a feature to the GA version 2015-06-01 on September 26, 2016.

General Availability (version 2015-06-01) was announced on June 24, 2015. The following changes were made at this release:
- Delete a key - "use" field removed.
- Get information about a key - "use" field removed.
- Import a key into a vault - "use" field removed.
- Restore a key - "use" field removed.
- Changed "RSA_OAEP" to "RSA-OAEP" for RSA Algorithms. See [About keys, secrets, and certificates](about-keys-secrets-certificates.md).

Second preview version (version 2015-02-01-preview) was announced April 20, 2015. For more information, see [REST API Update](/archive/blogs/kv/rest-api-update) blog post. The following tasks were updated:

- List the keys in a vault - added pagination support to operation.
- List the versions of a key - added operation to list the versions of a key.
- List secrets in a vault - added pagination support.
- List versions of a secret - add operation to list the versions of a secret.
- All operations - Added created/updated timestamps to attributes.
- Create a secret - added Content-Type to secrets.
- Create a key - added tags as optional information.
- Create a secret - added tags as optional information.
- Update a key - added tags as optional information.
- Update a secret - added tags as optional information.
- Changed max size for secrets from 10 K to 25 K Bytes. See [About keys, secrets, and certificates](about-keys-secrets-certificates.md).

## 2014

First preview version (version 2014-12-08-preview) was announced on January 8, 2015.

## Next steps

If you have questions, contact us through [support](https://azure.microsoft.com/support/options/).
