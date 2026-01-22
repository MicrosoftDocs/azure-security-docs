---
title: About Azure Key Vault Certificates access control
description: Overview of Azure Key Vault certificates access control
services: key-vault
author: sebansal

ms.service: azure-key-vault
ms.subservice: certificates
ms.topic: overview
ms.date: 01/08/2026
ms.author: sebansal
---

# Certificate access control

Key Vault manages access control for certificates at the key vault level. The access control policy for certificates is distinct from the access control policies for keys and secrets in the same key vault. You can create one or more vaults to hold certificates, to maintain scenario-appropriate segmentation and management of certificates.

Access to a key vault is controlled through two interfaces: the **control plane** and the **data plane**. Both planes use Microsoft Entra ID for authentication. For authorization, you can use either [Azure role-based access control (Azure RBAC)](../general/rbac-guide.md) (recommended) or [Key Vault access policies](../general/assign-access-policy.md) (legacy). For more information about authentication and authorization concepts, see [Authentication in Azure Key Vault](../general/authentication.md).

## Certificate permissions

Use the following permissions on a per-principal basis when configuring access to certificates. These permissions closely mirror the operations allowed on a certificate object:  

- Permissions for certificate management operations
  - **get**: Get the current certificate version, or any version of a certificate
  - **list**: List the current certificates, or versions of a certificate  
  - **update**: Update a certificate
  - **create**: Create a Key Vault certificate
  - **import**: Import certificate material into a Key Vault certificate
  - **delete**: Delete a certificate, its policy, and all of its versions  
  - **recover**: Recover a deleted certificate
  - **backup**: Back up a certificate in a key vault
  - **restore**: Restore a backed-up certificate to a key vault
  - **managecontacts**: Manage Key Vault certificate contacts  
  - **manageissuers**: Manage Key Vault certificate authorities/issuers
  - **getissuers**: Get a certificate's authorities/issuers
  - **listissuers**: List a certificate's authorities/issuers  
  - **setissuers**: Create or update a Key Vault certificate's authorities/issuers  
  - **deleteissuers**: Delete a Key Vault certificate's authorities/issuers  
 
- Permissions for privileged operations
  - **purge**: Purge (permanently delete) a deleted certificate

For more information, see [Certificate operations in the Key Vault REST API reference](/rest/api/keyvault).

## Grant access to certificates

You can grant access to certificates by using Azure RBAC (recommended) or Key Vault access policies (legacy).

### Use Azure RBAC (recommended)

Azure RBAC provides centralized access management and allows you to set permissions at different scope levels. For certificate operations, use one of the following built-in roles:

| Role | Description |
|------|-------------|
| **Key Vault Administrator** | Perform all data plane operations on a key vault and all objects in it. |
| **Key Vault Certificates Officer** | Perform any action on the certificates of a key vault, except managing permissions. |
| **Key Vault Certificate User** | Read entire certificate contents, including secret and key portion. |
| **Key Vault Reader** | Read metadata of key vaults and its certificates, keys, and secrets. Can't read sensitive values. |

For details on assigning roles, see [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](../general/rbac-guide.md).

### Use access policies (legacy)

To assign an access policy, see [Assign a Key Vault access policy](../general/assign-access-policy.md). For information on establishing permissions via REST API, see [Vaults - Update Access Policy](/rest/api/keyvault/keyvault/vaults/update-access-policy).

## Troubleshoot

You might see an error due to a missing access policy or role assignment. For example: `Error type : Access denied or user is unauthorized to create certificate`.

To resolve this error:
- If using Azure RBAC, verify the principal has a role with the `certificates/create` permission (such as **Key Vault Certificates Officer**).
- If using access policies, add the `certificates/create` permission to the access policy.

For more troubleshooting guidance, see [Troubleshooting Azure Key Vault access issues](../general/troubleshooting-access-issues.md).

## Next steps

- [Authentication in Azure Key Vault](../general/authentication.md)
- [Provide access to Key Vault keys, certificates, and secrets with Azure RBAC](../general/rbac-guide.md)
- [Assign a Key Vault access policy](../general/assign-access-policy.md)
- [Secure access to a key vault](../general/secure-key-vault.md)
- [About Key Vault](../general/overview.md)
- [About keys, secrets, and certificates](../general/about-keys-secrets-certificates.md)
- [Key Vault Developer's Guide](../general/developers-guide.md)
