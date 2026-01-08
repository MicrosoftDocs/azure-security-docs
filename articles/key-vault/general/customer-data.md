---
title: Customer data requests for Azure Key Vault
description: Learn how to identify, export, and delete personal data from Azure Key Vault to fulfill GDPR data subject requests.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: concept-article
ms.date: 01/08/2026
---
# Customer data requests for Azure Key Vault

This article describes how to identify, export, and delete personal data from Azure Key Vault to fulfill data subject requests under GDPR and similar privacy regulations.

[!INCLUDE [GDPR-related guidance](~/reusable-content/ce-skilling/azure/includes/gdpr-intro-sentence.md)]

## Identifying personal data

The following Key Vault data may contain personal information:

- **Access control data**: [Azure RBAC](rbac-guide.md) role assignments and legacy [access policies](assign-access-policy.md) contain object IDs representing users or groups
- **Certificate contacts and issuers**: May contain email addresses, names, or phone numbers
- **Resource tags**: [Tags](/azure/azure-resource-manager/management/tag-resources) applied to vaults or objects may contain personal data
- **Resource logs**: [Logs](logging.md) contain object IDs, [user principal names (UPNs)](/entra/identity/hybrid/connect/plan-connect-userprincipalname), and IP addresses

## Deleting personal data

To delete personal data, delete the Key Vault object containing the data, then purge it from the soft-deleted state.

[Soft-delete](soft-delete-overview.md) retains deleted objects for 7-90 days (default 90). To permanently delete data, [purge the deleted object](key-vault-recovery.md). If purge protection is enabled, data can't be deleted until the retention period expires.

## Exporting personal data

Export Key Vault data using the Azure portal, Azure CLI, Azure PowerShell, or the [REST API](/rest/api/keyvault/).

To export access logs containing personal data, configure [diagnostic settings](howto-logging.md) to send logs to a storage account you control.

> [!NOTE]
> Cryptographic key material marked as non-exportable cannot be exported in plaintext form.

## Related documentation

- [Azure Key Vault REST API](/rest/api/keyvault/)
- [Azure Key Vault soft-delete overview](soft-delete-overview.md)
- [Manage Key Vault recovery](key-vault-recovery.md)
- [Azure Key Vault logging](logging.md)