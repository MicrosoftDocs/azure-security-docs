---
title: User Management Best Practices in Azure Cloud HSM
description: Learn best practices for managing user identities, securing credentials, implementing redundancy, and restricting user permissions in Azure Cloud HSM.
author: msmbaldwin
ms.service: azure-dedicated-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ms.author: mbaldwin

# Customer intent: As a security administrator, I need to manage user identities and permissions in Azure Cloud HSM to ensure security and compliance.

---

# User management in Azure Cloud HSM

Effective user management is crucial for maintaining the security and integrity of Azure Cloud HSM. This article outlines best practices for managing user identities, securing credentials, implementing redundancy, and restricting user permissions to enhance security and prevent unauthorized access.

## User Managed Identity

Azure Cloud HSM customers can establish a user-assigned managed identity. However, Managed Identity for Azure Cloud HSM is tailored specifically for user backup/restore operations. This facilitates transferring Cloud HSM backups to the customer's storage account, enabling BCDR scenarios, including provisioning new Cloud HSM instances from existing backups. This setup is essential for customers needing access to their backups and the capability to independently execute restore operations to fulfill BCDR requirements.  

Microsoft suggests adopting user-assigned managed identity for Azure VMs utilizing the Azure Cloud HSM SDK (for example, Cloud HSM Client) or VMs where administrative actions are performed, or applications accessing HSM resources. This approach enhances security, simplifies management, ensures scalability, and aids in meeting compliance requirements. It provides a secure and traceable method for managing administrative access to Azure VMs.

## Use a Strong, Unique Password

We strongly recommend that you create unique, strong passwords. Azure Cloud HSM has a minimum password length of 8 characters, with a max password length of 32 characters. We recommend using a password that is at least 12 characters long.  

Incorporate a mix of uppercase and lowercase letters, numbers, and special characters. Avoid common words and personal information. Consider using a random phrase or sentence that is easy for you to remember but difficult for others to guess.

## Secure your HSM user credentials

Ensuring the secure protection of your HSM user credentials is paramount, as these credentials grant access to perform cryptographic and management operations on your HSM. Azure Cloud HSM does not retain access to your HSM user credentials. Therefore, if access is lost, Microsoft cannot aid if you lose access to your credentials.

## Implement Secondary Admins for Lockout Prevention

To prevent the risk of HSM lockout, it's advisable to designate at least two administrators, which ensures continuity in case one admin password is lost. In such an event, the other admin can be utilized to reset the password.

## Establish Multiple Crypto Users with Restricted Permissions

When responsibilities are distributed among crypto users, no single user possesses absolute control over the entire system. You should therefore create multiple crypto users and restrict the permissions of each accordingly. This often involves assigning distinct responsibilities and actions to different crypto users.  

For instance, one crypto user might be tasked with generating and distributing keys, while others utilize these keys within your application.

## Key Sharing with Multiple Crypto Users

Keys can be shared with other crypto users. However, only the original owner of the key can wrap it. Additional users can utilize the key but cannot perform a plain text export, even if they have access to a key marked as TRUSTED by the CO.

## Limiting Crypto User Ability to Export Keys

The CO can define a set of attributes to regulate the operations a Crypto User can execute. These attributes include restricting the CU's capability to wrap/unwrap keys and modify key attributes or derive keys. This limitation helps prevent the CU from extracting private key material from the HSM.  

The `setUserAttributes` command can be utilized to configure these attributes.

## Limiting CO Control Over Crypto Users

If there is a need to restrict CO users from accessing a CU's key material (for example, to prevent CO from both marking a key as trusted and performing a plaintext export using that key), it's possible to revoke CO's management access to a user.  

The `disableUserAccess` command achieves this by revoking CO's access to the specified crypto user. However, this measure could potentially be circumvented by using an earlier backup that lacks this restriction.

## Next steps

- [Secure your Azure Cloud HSM](secure-cloud-hsm.md)
- [Network security](network-security.md)
- [Key Management in Azure Cloud HSM](key-management.md)
- [User Management in Azure Cloud HSM](user-management.md)
- [Authentication in Azure Cloud HSM](authentication.md)
- [Overview of Azure Cloud HSM](overview.md)