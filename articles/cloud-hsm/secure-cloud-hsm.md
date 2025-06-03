---
title: Best Practices for Securing Microsoft Azure Cloud HSM
description: Learn the best practices for securing and managing Microsoft Azure Cloud HSM to help protect cryptographic keys and sensitive workloads.
author: msmbaldwin
ms.service: azure-cloud-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ai-usage: ai-assisted
ms.custom: horz-security
ms.author: mbaldwin

#customer intent: As a Cloud HSM administrator, I want to learn how to secure and optimize my Cloud HSM deployment.
---

# Secure your Azure Cloud HSM deployment

Microsoft Azure Cloud HSM provides a high-assurance hardware security module (HSM) solution for protecting cryptographic keys and secure sensitive workloads in the cloud. Implementing security best practices is essential to prevent unauthorized access, maintain operational integrity, and optimize performance.

This article provides guidance on how to best secure your Cloud HSM deployment.

## Security and compliance  

- **Protect root of trust**: We advise customers to limit access to the Partition Owner of the Application Partition (POTA) private key (also called *PO.key*). The Admin of the Application Partition (AOTA) and POTA private keys are equivalent to root access. They can reset passwords for cryptography officer users in a partition (AOTA for partition 0, POTA for user partitions).

  PO.key is unnecessary for HSM access during runtime. It's required only for the initial signing of Purchase Order Accruals (POAC) and Controlling (CO) password resets. We recommend storing PO.key offline and performing the initial POAC signing on an offline machine, if possible.

  > [!IMPORTANT]
  > Customers are accountable for safeguarding their POTA private key. Losing the POTA private key results in the inability to recover CO passwords. We advise customers to securely store their POTA private key and maintain suitable backups.

## Network security  

Ensuring strong network security is essential when you're using Azure Cloud HSM. Properly configuring your network can help prevent unauthorized access and reduce exposure to external threats. For more information, see [Network security for Azure Cloud HSM](network-security.md).  

- **Use private endpoints**: Help secure your Azure Cloud HSM deployment by using private subnets and private endpoints to prevent exposure to the public internet. This action ensures that traffic remains within the Microsoft backbone network, which reduces the risk of unauthorized access.

## User management  

Effective user management is crucial for maintaining the security and integrity of Azure Cloud HSM. Implementing proper controls for user identities, credentials, and permissions can help prevent unauthorized access and ensure operational continuity. For more information, see [User management in Azure Cloud HSM](user-management.md).  

- **Use strong passwords**: Create unique, strong passwords for HSM users. Use at least 12 characters, including a mix of uppercase and lowercase letters, numbers, and special characters.

- **Secure your HSM user credentials**: Protect your HSM user credentials carefully, because Microsoft can't recover them if they're lost.

- **Implement secondary admins for lockout prevention**: Designate at least two administrators to prevent HSM lockout in case one password is lost.

- **Establish multiple cryptography users with restricted permissions**: Create multiple cryptography users with distinct responsibilities to prevent any single user from having full control.

- **Limit the ability of cryptography users to export keys**: Restrict cryptography users from exporting key material by setting appropriate user attributes.

- **Limit CO control over cryptography users**: Use the `disableUserAccess` command to prevent CO users from managing specific cryptography users. However, this command can be bypassed with older backups.

## Key management  

Effective key management is critical for optimizing the performance, security, and efficiency of Azure Cloud HSM. Proper handling of key storage limits, key wrapping security, key attributes, and caching strategies can improve protection and performance. For more information, see [Key management in Azure Cloud HSM](key-management.md).  

- **Implement key rotation**: Regularly rotate keys to replace older ones and free up storage while maintaining security.  

- **Use a key hierarchy**: Store fewer keys in the HSM by using master keys to encrypt other keys.  

- **Share and reuse keys when feasible**: Reduce storage requirements by sharing or reusing keys across multiple sessions when appropriate.  

- **Securely delete unused keys**: Remove keys that you longer need, to prevent unnecessary storage consumption.  

- **Set keys as non-extractable when possible**: Use `EXTRACTABLE=0` to ensure that keys can't be exported outside the HSM.  

- **Enable trusted key wrapping**: Use `WRAP_WITH_TRUSTED=1` to restrict key wrapping to trusted keys. This action prevents unauthorized key exports.  

- **Use key attributes to restrict permissions**: Assign only necessary attributes when you're generating keys, to limit unintended operations.  

## Authentication  

Authentication is a crucial aspect of securely accessing and operating within Azure Cloud HSM. Proper authentication methods help protect credentials and ensure secure access control. For more information, see [Authentication in Azure Cloud HSM](authentication.md).  

- **Securely store HSM credentials**: Protect stored credentials and avoid exposing them when they're not in use. Configure your environment to retrieve and set credentials automatically.  

- **Use implicit login for Java Cryptography Extension (JCE) authentication**: Whenever possible, use implicit login for JCE authentication to allow automatic credential management and reauthentication.  

- **Avoid sharing sessions across threads**: For multithreaded applications, assign each thread its own session to prevent conflicts and security issues.  

- **Implement client-side retries**: Add retry logic for HSM operations to handle potential maintenance events or HSM replacements.  

- **Manage HSM client sessions carefully**: Be aware that `azurecloudhsm_client` shares sessions across applications on the same host. Proper session management avoids conflicts.  

## Monitoring and logging

- **Monitor audit and operations logs**: We recommend that you configure operation event logging. Operation event logging is vital for HSM security. It provides an immutable record of access and operations for accountability, traceability, and regulatory compliance. It helps detect unauthorized access, investigate incidents, and identify anomalies, to help ensure the integrity and confidentiality of cryptographic operations.  

  To maintain security and privacy, logs exclude sensitive data (such as key IDs, key names, and user details). They capture HSM operations, time stamps, and metadata, but they can't determine success or failure. They can only log the fact that the operation was executed. This limitation exists because the HSM operation occurs within the inner TLS channel, which is not exposed outside that boundary.  

## Business continuity and disaster recovery (BCDR)  

- **Implement robust backup and disaster recovery**: Azure Cloud HSM provides high availability through clustered HSMs that synchronize keys and policies while automatically migrating partitions during failures. The service supports comprehensive backup and restore operations that preserve all keys, attributes, and role assignments. Backups are secured by HSM-derived keys that Microsoft can't access.

  For disaster recovery:

  - Use managed service identities for authentication.
  - Store backups in private Azure Blob Storage.
  - Implement minimal role-based access control (RBAC) permissions.
  - Disable shared key access.

  > [!NOTE]
  > Azure Cloud HSM doesn't support restoring to already activated HSMs.

  For detailed implementation instructions and additional recovery options, see [Backup and restore in Azure Cloud HSM](backup-restore.md). Additional recovery options include using `extractMaskedObject` to extract keys as encrypted blobs, storing them securely, and importing them with `insertMaskedObject` as needed. A BCDR best practice is to deploy in two regions for failover capability.

## Related content

- [Security best practices for IaaS workloads in Azure](/azure/security/fundamentals/iaas)
- [Enable just-in-time access to virtual machines](/azure/defender-for-cloud/just-in-time-access-usage)
- [Adopt a Zero Trust approach](/azure/security/fundamentals/network-best-practices#adopt-a-zero-trust-approach)
