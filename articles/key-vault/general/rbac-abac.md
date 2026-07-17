---
title: Actions and attributes for Azure Key Vault ABAC conditions
description: Reference for the actions, attributes, and operators that you can use in Azure attribute-based access control (Azure ABAC) conditions on role assignments for Azure Key Vault.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-key-vault
ms.subservice: general
ms.topic: reference
ms.date: 06/12/2026
ms.custom: sfi-image-nochange
ai-usage: ai-assisted
---

# Actions and attributes for Azure Key Vault ABAC conditions (preview)

> [!NOTE]
> Azure ABAC for Key Vault is in preview. Some aspects might change before general availability. For preview terms, see the [supplemental terms of use](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

This article describes the supported actions, attributes, and operators that you can use in conditions on Azure role assignments for Azure Key Vault. For an overview of how role assignment conditions work, see [What is Azure attribute-based access control (Azure ABAC)?](/azure/role-based-access-control/conditions-overview). For the list of Key Vault data actions that a specific permission affects, see the [Azure RBAC for Key Vault data plane operations](rbac-guide.md) guide.

## Overview

Azure ABAC for Key Vault builds on Azure RBAC by adding [conditions to role assignments](/azure/role-based-access-control/conditions-overview). Azure evaluates each condition at runtime against the attributes of the resource being accessed or the incoming request, and either allows or denies the request based on comparison operators.

The benefits of using role assignment conditions for Key Vault are:

- **Finer-grained access to secrets.** Restrict a principal to secrets whose names match a specific pattern (for example, `app-*` or `team-prod-*`), or to specific key vaults by vault name.
- **Fewer role assignments to manage.** Use a single condition-gated role assignment for a security group instead of many individual assignments that model the same permissions. ABAC helps you scale role management - it doesn't grant permissions that a role assignment doesn't already grant.
- **Extends existing Azure RBAC.** ABAC layers on top of your existing role assignments. You don't have to replace RBAC to use it.

In this preview, ABAC for Key Vault applies **only to secret data actions** and supports **two attributes**: the vault name and the secret name. Key data actions, certificate data actions, tag-based attributes, token attributes, and built-in Key Vault ABAC conditions on any role aren't included.

To use ABAC conditions, the key vault's permission model must be set to Azure RBAC. For background, see [Azure RBAC vs. access policies](rbac-access-policy.md) and [Prepare for Key Vault API version 2026-02-01 and later](access-control-default.md).

## Condition format

Conditions use the following general format:

```
@Resource[<ResourceType>:<AttributeName>] <Operator> '<Value>'
```

| Component | Description |
| --- | --- |
| `@Resource` | Refers to an attribute of the resource being accessed. Use `@Request` when the attribute describes the incoming request (for example, the name of a secret that doesn't yet exist). |
| `ResourceType` | The Azure Resource Manager resource type. For Key Vault in this preview, either `Microsoft.KeyVault/vaults` or `Microsoft.KeyVault/vaults/secrets`. |
| `AttributeName` | The specific attribute to evaluate. In this preview, `name` is the only supported attribute. |
| `Operator` | The comparison operator. See [Supported operators](#supported-operators). |
| `Value` | The literal value to compare against. |

The condition version must be set to `2.0`. For the complete grammar, see [Azure role assignment condition format and syntax](/azure/role-based-access-control/conditions-format).

## Supported operators

All string comparison operators are supported in ABAC conditions for Azure Key Vault.

| Operator | Description | Case-sensitive |
| --- | --- | --- |
| `StringEquals` | Exact string equality. | Yes |
| `StringNotEquals` | String inequality. | Yes |
| `StringStartsWith` | String prefix match. | Yes |
| `StringNotStartsWith` | String doesn't start with value. | Yes |
| `StringLike` | Pattern match with wildcards (`*`, `?`). | Yes |
| `StringNotLike` | Pattern doesn't match. | Yes |
| `StringEndsWith` | String suffix match. | Yes |
| `StringNotEndsWith` | String doesn't end with value. | Yes |
| `StringContains` | String contains substring. | Yes |
| `StringNotContains` | String doesn't contain substring. | Yes |

For the full definition of each operator, see [String comparison operators](/azure/role-based-access-control/conditions-format#string-comparison-operators).

> [!NOTE]
> All string operators are case-sensitive. A condition with `StringStartsWith 'test-app'` doesn't match a secret named `Test-App-Secret`. Use consistent casing in your naming conventions.

## Azure Key Vault actions

You can apply ABAC conditions to the following Key Vault secret data actions. Each subsection describes the action, the underlying DataAction, the attribute source it evaluates (Resource or Request), and an example condition.

### Get secret

| Property | Value |
| --- | --- |
| Display name | Get secret |
| Description | Gets the value of a secret. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/getSecret/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'` |

### Read secret metadata

| Property | Value |
| --- | --- |
| Display name | Read secret metadata |
| Description | Lists or views the properties of a secret, but not its value. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/readMetadata/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/readMetadata/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'api-'` |

### Set secret

| Property | Value |
| --- | --- |
| Display name | Set secret |
| Description | Sets the value of a secret. If the secret doesn't exist, the first version is created. Otherwise, a new version is created with the specified value. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/setSecret/action` |
| Request attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/setSecret/action'}) OR @Request[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'app-'` |

### Update secret

| Property | Value |
| --- | --- |
| Display name | Update secret |
| Description | Updates the attributes associated with the given secret. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/update/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/update/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'app-'` |

### Rotate secret

| Property | Value |
| --- | --- |
| Display name | Rotate secret |
| Description | Rotates a secret based on the secret's rotation policy. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/rotate/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/rotate/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'rotating-'` |

### Delete secret

| Property | Value |
| --- | --- |
| Display name | Delete secret |
| Description | Deletes a secret. All versions are deleted. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/delete` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/delete'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'temp-'` |

### Backup secret

| Property | Value |
| --- | --- |
| Display name | Backup secret |
| Description | Creates the backup file of a secret. The file can be used to restore the secret in a key vault of the same subscription. Restrictions might apply. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/backup/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/backup/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'critical-'` |

### Restore secret

| Property | Value |
| --- | --- |
| Display name | Restore secret |
| Description | Restores a secret and all its versions from a backup file generated by Key Vault. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/restore/action` |
| Request attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/restore/action'}) OR @Request[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'app-'` |

### Recover secret

| Property | Value |
| --- | --- |
| Display name | Recover secret |
| Description | Recovers a deleted secret. The operation performs the reversal of the Delete operation and is applicable in vaults enabled for soft-delete. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/recover/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/recover/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'app-'` |

### Purge secret

| Property | Value |
| --- | --- |
| Display name | Purge secret |
| Description | Purges a deleted secret, making it unrecoverable. |
| DataAction | `Microsoft.KeyVault/vaults/secrets/purge/action` |
| Resource attributes | Secret name |
| Example | `!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/purge/action'}) OR @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'old-'` |

## Attribute sources

Attributes describe properties evaluated at request time. Resource attributes describe the resource being accessed. Request attributes describe the incoming request when the resource might not yet exist (for example, during `setSecret` or `restore`).

### Vault name

Use this attribute to scope access to key vaults whose names match a given pattern. When you apply a condition at subscription or resource group scope, Azure evaluates the vault name for every vault the principal attempts to access.

| Property | Value |
| --- | --- |
| Display name | Vault name |
| Description | The name of the key vault being accessed. |
| Attribute | `Microsoft.KeyVault/vaults:name` |
| Attribute source | Resource |
| Attribute type | String |
| Applicable actions | All Key Vault secret data actions (global attribute) |

Example conditions:

```
@Resource[Microsoft.KeyVault/vaults:name] StringStartsWith 'prod-'
@Resource[Microsoft.KeyVault/vaults:name] StringEquals 'sample-vault'
@Resource[Microsoft.KeyVault/vaults:name] StringLike 'team-*-prod'
```

### Secret name (Resource)

Use this attribute when the secret already exists at evaluation time.

| Property | Value |
| --- | --- |
| Display name | Secret name |
| Description | The name of the secret being accessed. |
| Attribute | `Microsoft.KeyVault/vaults/secrets:name` |
| Attribute source | Resource |
| Attribute type | String |
| Applicable actions | `getSecret/action`, `readMetadata/action`, `update/action`, `rotate/action`, `delete`, `backup/action`, `recover/action`, `purge/action` |

Example conditions:

```
@Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'
@Resource[Microsoft.KeyVault/vaults/secrets:name] StringLike 'api-key-*'
```

### Secret name (Request)

Use this attribute when the secret might not yet exist at evaluation time, such as during creation or restore.

| Property | Value |
| --- | --- |
| Display name | Secret name (Request) |
| Description | The name of a secret being requested. |
| Attribute | `Microsoft.KeyVault/vaults/secrets:name` |
| Attribute source | Request |
| Attribute type | String |
| Applicable actions | `setSecret/action`, `restore/action` |

Example conditions:

```
@Request[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'app-'
@Request[Microsoft.KeyVault/vaults/secrets:name] StringLike 'config-*'
```

## Roles that support ABAC conditions

You can apply ABAC conditions to any role that includes Key Vault secret `dataActions`. Roles that only contain control-plane actions (such as Owner or Policy Administrator) don't support ABAC conditions because Azure evaluates conditions exclusively on data-plane operations. In this preview, no built-in role includes a preassigned Key Vault ABAC condition - you author the condition when you create the role assignment.

The following built-in roles include secret data actions and support ABAC conditions in this preview. For the full role definitions and IDs, see [Azure built-in roles for Key Vault data plane operations](rbac-guide.md#azure-built-in-roles-for-key-vault-data-plane-operations).

| Role | Description | ID | Condition support |
| --- | --- | --- | --- |
| Key Vault Administrator | Perform all data plane operations on a key vault and all objects in it. | `00482a5a-887f-4fb3-b363-3b7fe8e74483` | Resource and Request attributes on secret operations |
| Key Vault Secrets Officer | Perform any action on secrets. | `b86a8fe4-44ce-4948-aee5-eccb2c155cd7` | Resource attributes on read and lifecycle operations; Request attributes on `setSecret` and `restore` operations |
| Key Vault Secrets User | Read secret contents. | `4633458b-17de-408a-b874-0445c86b69e6` | Resource attributes on secret read operations |
| Key Vault Reader | Read metadata of vaults and their objects. | `21090545-7ca7-4776-b22c-e363652d74d2` | Resource attributes on read and list operations |
| Key Vault Certificate User | Read certificate contents. | `db79e9a7-68ee-4b58-9aeb-b90e7c24fcba` | Resource attributes on `getSecret` and `readMetadata` (certificates are retrieved through the secret endpoint - the secret name equals the certificate name) |
| Custom roles | Any custom role definition that includes secret `dataActions` for Key Vault. | - | Depends on which secret `dataActions` are included |

> [!NOTE]
> Only roles assigned through Azure RBAC (not legacy vault access policies) support ABAC conditions. The key vault must have its permission model set to Azure role-based access control. Starting with API version 2026-02-01, Azure RBAC is the default for newly created vaults. For more information, see [Prepare for Key Vault API version 2026-02-01 and later](access-control-default.md).

## Condition examples

The following examples show common ABAC patterns for Key Vault secrets. For more on the condition syntax, see [Azure role assignment condition format and syntax](/azure/role-based-access-control/conditions-format).

### Example: Restrict read access to secrets with a name prefix

Allow a principal to read only secrets whose names start with `test-app`:

- **Role**: Key Vault Secrets User
- **Scope**: Key vault resource
- **Condition**:

```
(
  !(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})
  OR
  (
    @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'test-app'
  )
)
```

Behavior:

- Allowed: Reading `test-app-secret1`.
- Denied (`ForbiddenByRbac`): Reading `test-db-password`.
- Denied (case-sensitive match): Reading `Test-App-Secret1`.

### Example: Restrict access to secrets in specific vaults by vault name prefix

Allow a principal to read secrets only in vaults whose names start with `kvsyn`:

- **Role**: Key Vault Secrets User
- **Scope**: Subscription
- **Condition**:

```
@Resource[Microsoft.KeyVault/vaults:name] StringStartsWith 'kvsyn'
```

Behavior:

- Allowed: Access to `kvsynabaceuap0520`.
- Denied: Access to `kvdenyabaceuap0520b`.

### Example: Restrict newly created secrets to a name prefix

Allow a principal to create secrets only when the secret name starts with `app-`. Because the secret doesn't yet exist at evaluation time, use the Request attribute:

- **Role**: Key Vault Secrets Officer
- **Scope**: Key vault resource
- **Condition**:

```
(
  !(ActionMatches{'Microsoft.KeyVault/vaults/secrets/setSecret/action'})
  OR
  (
    @Request[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'app-'
  )
)
```

### Example: Combine vault and secret name conditions

Restrict a principal to read only secrets that start with `api-` in vaults whose names start with `kv-prod`:

- **Role**: Key Vault Secrets User
- **Scope**: Subscription
- **Condition**:

```
(
  !(ActionMatches{'Microsoft.KeyVault/vaults/secrets/getSecret/action'})
  OR
  (
    @Resource[Microsoft.KeyVault/vaults:name] StringStartsWith 'kv-prod'
    AND
    @Resource[Microsoft.KeyVault/vaults/secrets:name] StringStartsWith 'api-'
  )
)
```

## Related content

- [Add role assignment conditions for Key Vault secrets by using Azure CLI (preview)](howto-abac-conditions-cli.md)
- [Add role assignment conditions for Key Vault secrets by using Azure PowerShell (preview)](howto-abac-conditions-powershell.md)
- [What is Azure attribute-based access control (Azure ABAC)?](/azure/role-based-access-control/conditions-overview)
- [Azure role assignment condition format and syntax](/azure/role-based-access-control/conditions-format)
- [Provide access to Key Vault keys, certificates, and secrets with Azure RBAC](rbac-guide.md)
- [Troubleshoot Azure role assignment conditions](/azure/role-based-access-control/conditions-troubleshoot)
- [Troubleshoot Key Vault access issues](troubleshooting-access-issues.md)
- [Prepare for Key Vault API version 2026-02-01 and later](access-control-default.md)
