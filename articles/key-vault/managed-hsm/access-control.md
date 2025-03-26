---
title: Azure Key Vault Managed HSM access control
description: Learn how to manage access permissions for Azure Key Vault Managed HSM and keys. Understand the authentication and authorization models for Managed HSM and how to secure your HSMs.
services: key-vault
author: msmbaldwin

ms.service: azure-key-vault
ms.subservice: managed-hsm
ms.topic: concept-article
ms.date: 01/30/2024
ms.author: mbaldwin

# Customer intent: As the admin for managed HSMs, I want to set access policies and configure the Managed HSM, so that I can ensure it's secure and auditors can properly monitor all activities for these managed HSMs.
---

# Access control for Managed HSM

Azure Key Vault Managed HSM is a cloud service that safeguards encryption keys. Because this data is sensitive and critical to your business, you need to secure your managed hardware security modules (HSMs) by allowing only authorized applications and users to access the data.

This article provides an overview of the Managed HSM access control model. It explains authentication and authorization, and describes how to secure access to your managed HSMs.

> [!NOTE]
> The Azure Key Vault resource provider supports two resource types: *vaults* and *managed HSMs*. Access control that's described in this article applies only to *managed HSMs*. To learn more about access control for Managed HSM, see [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](../general/rbac-guide.md).

## Access control model

Access to a managed HSM is controlled through two interfaces:

- Management plane
- Data plane

On the management plane, you manage the HSM itself. Operations in this plane include creating and deleting managed HSMs and retrieving managed HSM properties.

On the data plane, you work with the data that's stored in a managed HSM. That is, you work with the HSM-backed encryption keys. You can add, delete, modify, and use keys to perform cryptographic operations, manage role assignments to control access to the keys, create a full HSM backup, restore a full backup, and manage the security domain from the data plane interface.

To access a managed HSM in either plane, all callers must have proper authentication and authorization. *Authentication* establishes the identity of the caller. *Authorization* determines which operations the caller can execute. A caller can be any one of the [security principals](/azure/role-based-access-control/overview#security-principal) that are defined in Microsoft Entra ID: user, group, service principal, or managed identity.

Both planes use Microsoft Entra ID for authentication. For authorization, they use different systems:

- The management plane uses Azure role-based access control (Azure RBAC), an authorization system that's built on Azure Resource Manager.
- The data plane uses a managed HSM-level RBAC (Managed HSM local RBAC), an authorization system that's implemented and enforced at the managed HSM level.

When a managed HSM is created, the requestor provides a list of data plane administrators (all [security principals](/azure/role-based-access-control/overview#security-principal) are supported). Only these administrators can access the managed HSM data plane to perform key operations and manage data plane role assignments (Managed HSM local RBAC).

The permissions models for both planes use the same syntax, but they're enforced at different levels, and role assignments use different scopes. Management plane Azure RBAC is enforced by Azure Resource Manager, and data plane Managed HSM local RBAC is enforced by the managed HSM itself.

> [!IMPORTANT]
> Granting management plane access to a security principal does *not* grant the security principal data plane access. For example, a security principal with management plane access doesn't automatically have access to keys or data plane role assignments. This isolation is by design, to prevent inadvertent expansion of privileges that affect access to keys that are stored in Managed HSM.
>
> But there's an exception: Members of the Microsoft Entra Global Administrator role can always add users to the Managed HSM Administrator role for recovery purposes, such as when there are no longer any valid Managed HSM Administrator accounts. For more information, see [Microsoft Entra ID best practices for securing the Global Adminstrator role](/azure/active-directory/roles/best-practices#5-limit-the-number-of-global-administrators-to-less-than-5).

For example, a subscription administrator (because they have Contributor permissions to all resources in the subscription) can delete a managed HSM in their subscription. But if they don't have data plane access specifically granted through Managed HSM local RBAC, they can't gain access to keys or manage role assignments in the managed HSM to grant themselves or others access to the data plane.

### Role-based access control for Managed HSM

The following table summarizes the role assignments for common scenarios:

| Role | Management plane role | Data plane role |
| --- | --- | --- |
| Security team | Managed HSM Contributor | Managed HSM Administrator |
| Developers and operators | None | None |
| Auditors | None | Managed HSM Crypto Auditor |
| Managed identity of the VM used by the application | None | Managed HSM Crypto User |
| Managed identity of the storage account used by the application | None | Managed HSM Service Encryption |

#### Management plane and Azure RBAC

In the management plane, you use Azure RBAC to authorize the operations that a caller can execute. For example, the `Managed HSM Contributor` role allows users to create and manage managed HSMs. You can assign roles at different scopes, such as subscription, resource group, or specific resource levels.

#### Data plane and Managed HSM local RBAC

In the data plane, Managed HSM local RBAC is used to control access to keys and other data. Roles such as `Managed HSM Administrator`, `Managed HSM Crypto User`, and `Managed HSM Crypto Auditor` define specific permissions for accessing and managing keys.

For more information on Azure RBAC and Managed HSM local RBAC roles, see [Azure RBAC: Built-in roles](/azure/role-based-access-control/built-in-roles) and [Local RBAC built-in roles for Managed HSM](built-in-roles.md).

### Microsoft Entra Privileged Identity Management (PIM)

To enhance the security of administrative roles, use [Microsoft Entra Privileged Identity Management (PIM)](/entra/id-governance/privileged-identity-management/pim-configure). PIM enables just-in-time access, reducing the risk of standing administrative privileges. It also provides visibility into role assignments and enforces approval workflows for elevated access.

## Microsoft Entra authentication

When you create a managed HSM in an Azure subscription, the managed HSM is automatically associated with the Microsoft Entra tenant of the subscription. All callers in both planes must be registered in this tenant and authenticate to access the managed HSM.

The application authenticates with Microsoft Entra ID before calling either plane. The application can use any [supported authentication method](/azure/active-directory/develop/authentication-vs-authorization) depending on the application type. The application acquires a token for a resource in the plane to gain access. The resource is an endpoint in the management plane or data plane, depending on the Azure environment. The application uses the token and sends a REST API request to the managed HSM endpoint. To learn more, review the entire [authentication flow](/azure/active-directory/develop/v2-oauth2-auth-code-flow).

Using a single authentication mechanism for both planes has several benefits:

- Organizations can centrally control access to all managed HSMs in their organization.
- If a user leaves the organization, they instantly lose access to all managed HSMs in the organization.
- Organizations can customize authentication by using options in Microsoft Entra ID, such as to enable multi-factor authentication for added security.

## Resource endpoints

Security principals access the planes through endpoints. The access controls for the two planes work independently. To grant an application access to use keys in a managed HSM, you grant data plane access by using Managed HSM local RBAC. To grant a user access to Managed HSM resource to create, read, delete, move the managed HSMs and edit other properties and tags, you use Azure RBAC.

The following table shows the endpoints for the management plane and data plane.

| Access plane | Access endpoints | Operations | Access control mechanism |
| --- | --- | --- | --- |
| Management plane | **Global:**<br/> `management.azure.com:443`<br/> | Create, read, update, delete, and move managed HSMs<br/><br/>Set managed HSM tags | Azure RBAC |
| Data plane | **Global:**<br/> `<hsm-name>.managedhsm.azure.net:443`<br/> | **Keys**: Decrypt, encrypt,<br/> unwrap, wrap, verify, sign, get, list, update, create, import, delete, back up, restore, purge<br/><br/> **Data plane role-management (Managed HSM local RBAC)**: List role definitions, assign roles, delete role assignments, define custom roles<br/><br/>**Backup and restore**: Back up, restore, check the status of backup and restore operations <br/><br/>**Security domain**: Download and upload the security domain | Managed HSM local RBAC |
|||||

## Data plane and Managed HSM local RBAC

You grant a security principal access to execute specific key operations by assigning a role. For each role assignment, you must specify a role and scope for which that assignment applies. For Managed HSM local RBAC, two scopes are available:

- **`/` or `/keys`**: HSM-level scope. Security principals that are assigned a role at this scope can perform the operations that are defined in the role for all objects (keys) in the managed HSM.
- **`/keys/<key-name>`**: Key-level scope. Security principals that are assigned a role at this scope can perform the operations that are defined in this role for all versions of the specified key only.

## Next steps

- For a getting-started tutorial for an administrator, see [Secure access to your managed HSMs](secure-your-managed-hsm.md).
- For more information about usage logging for Managed HSM, see [Managed HSM logging](logging.md).

