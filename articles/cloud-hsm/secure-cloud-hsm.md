---
title: Secure your Azure Cloud HSM deployment
description: Learn best practices for securing Azure Cloud HSM to help protect cryptographic keys and sensitive workloads.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/31/2026
ai-usage: ai-assisted
ms.custom: horz-security
ms.author: mbaldwin

#customer intent: As a Cloud HSM administrator, I want to learn how to secure and optimize my Cloud HSM deployment.
---

# Secure your Azure Cloud HSM deployment

Azure Cloud HSM provides a high-assurance hardware security module (HSM) solution for protecting cryptographic keys and securing sensitive workloads in the cloud. Implementing security best practices is essential to prevent unauthorized access, maintain operational integrity, and optimize performance.

This article provides security recommendations to help protect your Azure Cloud HSM deployment.

[!INCLUDE [Security horizontal Zero Trust statement](~/reusable-content/ce-skilling/azure/includes/security/zero-trust-security-horizontal.md)]

## Service-specific security

Azure Cloud HSM is a single-tenant, FIPS 140-3 Level 3 validated service that grants customers complete administrative authority over their HSMs. Protecting the root of trust is critical to maintaining the security of your deployment.

- **Restrict access to the Partition Owner private key**: Limit access to the Partition Owner of the Application Partition (POTA) private key (`PO.key`). The Admin of the Application Partition (AOTA) and POTA private keys are equivalent to root access and can reset passwords for cryptography officer (CO) users in a partition (AOTA for partition 0, POTA for user partitions). `PO.key` is unnecessary for HSM access during runtime. It's required only for the initial signing of Partition Owner Authentication Certificate (POAC) and CO password resets. Store `PO.key` offline and perform the initial POAC signing on an offline machine, if possible. Customers are accountable for safeguarding their POTA private key — losing it results in the inability to recover CO passwords. Securely store the POTA private key and maintain suitable backups. See [User management in Azure Cloud HSM](user-management.md).

## Network security

Properly configuring your network can help prevent unauthorized access and reduce exposure to external threats.

- **Use private endpoints**: Help secure your Azure Cloud HSM deployment by using private subnets and private endpoints to prevent exposure to the public internet. This approach ensures that traffic remains within the Microsoft backbone network, which reduces the risk of unauthorized access. See [Network security for Azure Cloud HSM](network-security.md).

## Identity and access management

Effective user management and authentication controls are crucial for maintaining the security and integrity of Azure Cloud HSM. Implementing proper controls for user identities, credentials, and permissions can help prevent unauthorized access and ensure operational continuity.

- **Use strong passwords**: Create unique, strong passwords for HSM users. Use at least 12 characters, including a mix of uppercase and lowercase letters, numbers, and special characters. See [User management in Azure Cloud HSM](user-management.md#use-strong-passwords).

- **Secure your HSM user credentials**: Protect your HSM user credentials carefully, because Microsoft can't recover them if they're lost. See [User management in Azure Cloud HSM](user-management.md#secure-your-hsm-user-credentials).

- **Implement secondary admins for lockout prevention**: Designate at least two administrators to prevent HSM lockout in case one password is lost. See [User management in Azure Cloud HSM](user-management.md#implement-secondary-admins-for-lockout-prevention).

- **Establish multiple cryptography users (CUs) with restricted permissions**: Create multiple CUs with distinct responsibilities to prevent any single user from having full control. See [User management in Azure Cloud HSM](user-management.md#establish-multiple-cryptography-users-with-restricted-permissions).

- **Limit the ability of CUs to export keys**: Restrict CUs from exporting key material by setting appropriate user attributes. See [User management in Azure Cloud HSM](user-management.md#limit-the-ability-of-cryptography-users-to-export-keys).

- **Limit CO control over CUs**: Use the `disableUserAccess` command to prevent CO users from managing specific CUs. CO users can bypass this command with older backups. See [User management in Azure Cloud HSM](user-management.md#limit-cryptography-officer-control-over-cryptography-users).

- **Configure managed identities**: Establish user-assigned managed identities for backup and restore operations and for VMs that perform administrative actions. See [User management in Azure Cloud HSM](user-management.md#establish-a-user-managed-identity).

- **Securely store HSM credentials**: Protect stored credentials and avoid exposing them when they're not in use. Configure your environment to retrieve and set credentials automatically. See [Authentication in Azure Cloud HSM](authentication.md).

- **Use implicit login for JCE authentication**: Whenever possible, use implicit login for Java Cryptography Extension (JCE) authentication to allow automatic credential management and reauthentication. See [Authentication in Azure Cloud HSM](authentication.md#jce-authentication).

- **Avoid sharing sessions across threads**: For multithreaded applications, assign each thread its own session to prevent conflicts and security issues. See [Authentication in Azure Cloud HSM](authentication.md#multithreading-techniques).

- **Implement client-side retries**: Add retry logic for HSM operations to handle potential maintenance events or HSM replacements. See [Authentication in Azure Cloud HSM](authentication.md#retries-for-integration-of-hsm-operations).

- **Manage HSM client sessions carefully**: Be aware that `azurecloudhsm_client` shares sessions across applications on the same host. Proper session management avoids conflicts. See [Authentication in Azure Cloud HSM](authentication.md#cloud-hsm-client-session-handling).

## Data protection

Proper handling of key storage limits, key wrapping security, key attributes, and caching strategies can improve protection and performance.

- **Implement key rotation**: Regularly rotate keys to replace older ones and free up storage while maintaining security. See [Key management in Azure Cloud HSM](key-management.md#manage-the-key-storage-limit).

- **Use a key hierarchy**: Store fewer keys in the HSM by using master keys to encrypt other keys. See [Key management in Azure Cloud HSM](key-management.md#manage-the-key-storage-limit).

- **Share and reuse keys when feasible**: Reduce storage requirements by sharing or reusing keys across multiple sessions when appropriate. See [Key management in Azure Cloud HSM](key-management.md#manage-the-key-storage-limit).

- **Securely delete unused keys**: Remove keys that you no longer need, to prevent unnecessary storage consumption. See [Key management in Azure Cloud HSM](key-management.md#manage-the-key-storage-limit).

- **Set keys as nonextractable when possible**: Use `EXTRACTABLE=0` to ensure that keys can't be exported outside the HSM. See [Key management in Azure Cloud HSM](key-management.md#manage-key-wrapping).

- **Enable trusted key wrapping**: Use `WRAP_WITH_TRUSTED=1` to restrict key wrapping to trusted keys. This action prevents unauthorized key exports. See [Key management in Azure Cloud HSM](key-management.md#manage-key-wrapping).

- **Use key attributes to restrict permissions**: Assign only necessary attributes when you're generating keys, to limit unintended operations. See [Key management in Azure Cloud HSM](key-management.md#employ-key-attributes-to-manage-key-permissions).

- **Cache key objects for performance**: Use key-finding commands only once during application startup and store the returned key object in application memory to reduce latency. See [Key management in Azure Cloud HSM](key-management.md#optimize-latency-by-caching-key-objects).

## Logging and monitoring

- **Configure operation event logging**: Operation event logging is vital for HSM security. It provides an immutable record of access and operations for accountability, traceability, and regulatory compliance. It helps detect unauthorized access, investigate incidents, and identify anomalies, to help ensure the integrity and confidentiality of cryptographic operations. To maintain security and privacy, logs exclude sensitive data (such as key IDs, key names, and user details). They capture HSM operations, timestamps, and metadata, but they can't determine success or failure because the HSM operation occurs within the inner TLS channel. See [Tutorial: Operation event logging in Azure Cloud HSM](tutorial-operation-event-logging.md).

## Backup and recovery

Azure Cloud HSM provides high availability through clustered HSMs that synchronize keys and policies while automatically migrating workloads during failures.

- **Implement robust backup and disaster recovery**: The service supports comprehensive backup and restore operations that preserve all keys, attributes, and role assignments. Backups are secured by HSM-derived keys that Microsoft can't access. Azure Cloud HSM doesn't support restoring to already activated HSMs. For business continuity, use managed identities for authentication, store backups in private Azure Blob Storage, implement minimal role-based access control (RBAC) permissions, and disable shared key access. Additional recovery options include using `extractMaskedObject` to extract keys as encrypted blobs, storing them securely, and importing them with `insertMaskedObject` as needed. Deploy in two regions for failover capability. See [Back up and restore Azure Cloud HSM resources](backup-restore.md).

- **Verify user and key synchronization across all nodes**: Azure Cloud HSM operates as a cluster of three nodes. During service events such as self-healing or upgrades, users or keys might not be replicated to all nodes. If you experience intermittent authentication or operation failures, verify that all users and keys exist on every node. See [Synchronize users and keys across Azure Cloud HSM nodes](synchronize-users-keys.md).

## Next steps

- [Security best practices for IaaS workloads in Azure](/azure/security/fundamentals/iaas)
- [Enable just-in-time access to virtual machines](/azure/defender-for-cloud/just-in-time-access-overview)
- [Adopt a Zero Trust approach](/azure/security/fundamentals/network-best-practices#adopt-a-zero-trust-approach)
- [Zero Trust guidance center](/security/zero-trust/zero-trust-overview)
