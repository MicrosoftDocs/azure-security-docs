---
title: User Management Best Practices in Azure Cloud HSM
description: Learn best practices for managing user identities, securing credentials, implementing redundancy, and restricting user permissions in Azure Cloud HSM.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin

#customer intent: As a security administrator, I need to manage user identities and permissions in Azure Cloud HSM so that I can ensure security and compliance.

---

# User management in Azure Cloud HSM

Effective user management is crucial for maintaining the security and integrity of Azure Cloud HSM. This article outlines best practices for managing user identities, securing credentials, implementing redundancy, and restricting user permissions to enhance security and prevent unauthorized access.

## Establish a user-managed identity

You can establish a user-assigned managed identity for Azure Cloud HSM. A managed identity for Azure Cloud HSM is tailored specifically for user backup and restore operations.

A managed identity facilitates transferring Cloud HSM backups to a storage account. It also enables business continuity and disaster recovery (BCDR) scenarios, including provisioning new Cloud HSM instances from existing backups. This setup is essential for customers who need access to their backups and the capability to independently execute restore operations to fulfill BCDR requirements.

We recommend that you adopt a user-assigned managed identity for Azure virtual machines (VMs) by using one of these items:

- The Azure Cloud HSM SDK (for example, the Cloud HSM client)
- VMs where administrative actions are performed
- Applications that access hardware security module (HSM) resources

This approach enhances security, simplifies management, helps ensure scalability, and aids in meeting compliance requirements. It provides a secure and traceable method for managing administrative access to Azure VMs.

## Use strong passwords

We recommend that you create unique, strong passwords. Azure Cloud HSM has a minimum password length of 8 characters, with a maximum length of 32 characters. We recommend using a password that has at least 12 characters.  

Incorporate a mix of uppercase and lowercase letters, numbers, and special characters. Avoid common words and personal information. Consider using a random phrase or sentence that's easy for you to remember but difficult for others to guess.

## Secure your HSM user credentials

Protecting your HSM user credentials is paramount, because these credentials grant access to perform cryptographic and management operations on your HSM.

Azure Cloud HSM does not retain access to your HSM user credentials. If you lose access to your credentials, Microsoft can't help.

## Implement secondary admins for lockout prevention

To prevent the risk of HSM lockout, we recommend that you designate at least two administrators to provide continuity. If one admin password is lost, the other admin can reset it.

## Establish multiple cryptography users with restricted permissions

When responsibilities are distributed among cryptography users (CUs), no single user has absolute control over the entire system. You should create multiple CUs and restrict the permissions of each accordingly. This task often involves assigning distinct responsibilities and actions to CUs.

For instance, one CU might be tasked with generating and distributing keys, whereas others use these keys within your application.

## Set up key sharing with multiple cryptography users

Keys can be shared with other CUs. However, only the original owner of the key can wrap it. Additional users can use the key but can't perform a plaintext export, even if they have access to a key that the cryptography officer (CO) user marked as `TRUSTED`.

## Limit the ability of cryptography users to export keys

The CO user can define a set of attributes to regulate the operations that a CU can execute. These attributes include restricting the CU's capability to wrap/unwrap keys and modify key attributes or derive keys. This limitation helps prevent the CU from extracting private key material from the HSM.

You can use the `setUserAttributes` command to configure these attributes.

## Limit cryptography officer control over cryptography users

If you need to restrict CO users from accessing a CU's key material, it's possible to revoke a CO user's management access. For example, you might want to prevent a CO user from both marking a key as trusted and performing a plaintext export by using that key.

The `disableUserAccess` command achieves this restriction by revoking a CO user's access to the specified CU. However, a CO user could potentially circumvent this measure by using an earlier backup that lacks this restriction.

## Related content

- [Secure your Azure Cloud HSM deployment](secure-cloud-hsm.md)
- [Network security in Azure Cloud HSM](network-security.md)
- [Key management in Azure Cloud HSM](key-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)
