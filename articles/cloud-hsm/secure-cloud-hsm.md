---
title: Best Practices for Securing Microsoft Azure Cloud HSM
description: Learn the best practices for securing and managing Microsoft Azure Cloud HSM to protect cryptographic keys and sensitive workloads.
author: msmbaldwin
ms.service: azure-dedicated-hsm
ms.topic: conceptual
ms.date: 03/20/2025
ai-usage: ai-assisted
ms.custom: horz-security
ms.author: mbaldwin

#Customer intent: As a Cloud HSM administrator, I want to learn how to secure and optimize my Cloud HSM deployment.
---

# Secure your Azure Cloud HSM

Microsoft Azure Cloud HSM provides a high-assurance hardware security module (HSM) solution for protecting cryptographic keys and securing sensitive workloads in the cloud. Implementing security best practices is essential to prevent unauthorized access, maintain operational integrity, and optimize performance.

This article provides guidance on how to best secure your Cloud HSM deployment.

## Security and compliance  

- **Protect Root of Trust**: Microsoft advises customers to limit access to the POTA private key (PO.key). The AOTA/POTA private keys are equivalent to root access and can reset passwords for crypto officer users in a partition (AOTA for partition 0, POTA for user partitions). The PO.key is unnecessary for HSM access during runtime; it's only required for the initial signing of POAC and CO password resets if necessary. Microsoft suggests storing the PO.key offline and performing the initial POAC signing onto an offline machine, if possible.  

    > [!IMPORTANT]
    > Customers are accountable for safeguarding their POTA private key (PO.key). Losing the POTA private key results in the inability to execute CO password recovery. Microsoft advises customers to securely store their POTA private key (PO.key) and maintain suitable backups.  

## Network security  

Ensuring strong network security is essential when using Azure Cloud HSM. Properly configuring your network can help prevent unauthorized access and reduce exposure to external threats. For more information, see [Network Security](network-security.md).  

- **Use private endpoints**: Secure your Azure Cloud HSM by using private subnets and private endpoints to prevent exposure to the public internet. This ensures traffic remains within the Microsoft backbone network, reducing the risk of unauthorized access.

## User management  

Effective user management is crucial for maintaining the security and integrity of Azure Cloud HSM. Implementing proper controls for user identities, credentials, and permissions can help prevent unauthorized access and ensure operational continuity. For more information, see [User Management in Azure Cloud HSM](user-management.md).  

- **Use a strong, unique password**: Create unique, strong passwords for HSM users. Use at least 12 characters, including a mix of uppercase and lowercase letters, numbers, and special characters.

- **Secure your HSM user credentials**: Protect your HSM user credentials carefully, as Microsoft cannot recover them if lost.

- **Implement secondary admins for lockout prevention**: Designate at least two administrators to prevent HSM lockout in case one password is lost.

- **Establish multiple crypto users with restricted permissions**: Create multiple crypto users with distinct responsibilities to prevent any single user from having full control.

- **Limit crypto user ability to export keys**: Restrict crypto users from exporting key material by setting appropriate user attributes.

- **Limit CO control over crypto users**: Use the `disableUserAccess` command to prevent CO users from managing specific crypto users, though this may be bypassed with older backups.

## Key management  

Effective key management is critical for optimizing the performance, security, and efficiency of Azure Cloud HSM. Proper handling of key storage limits, key wrapping security, key attributes, and caching strategies ensures better protection and performance. For more information, see [Key Management in Azure Cloud HSM](key-management.md).  

- **Implement key rotation**: Regularly rotate keys to replace older ones and free up storage while maintaining security.  

- **Use a key hierarchy**: Store fewer keys in the HSM by using master keys to encrypt other keys.  

- **Share and reuse keys when feasible**: Reduce storage requirements by sharing or reusing keys across multiple sessions when appropriate.  

- **Securely delete unused keys**: Remove keys no longer needed to prevent unnecessary storage consumption.  

- **Set keys as non-extractable when possible**: Use `EXTRACTABLE=0` to ensure keys cannot be exported outside the HSM.  

- **Enable trusted key wrapping**: Use `WRAP_WITH_TRUSTED=1` to restrict key wrapping to trusted keys, preventing unauthorized key exports.  

- **Use key attributes to restrict permissions**: Assign only necessary attributes when generating keys to limit unintended operations.  

## Authentication  

Authentication is a crucial aspect of securely accessing and operating within Azure Cloud HSM. Proper authentication methods help protect credentials and ensure secure access control. For more information, see [Authentication in Azure Cloud HSM](authentication.md).  

- **Securely store HSM credentials**: Protect stored credentials and avoid exposing them when not in use. Configure your environment to retrieve and set credentials automatically.  

- **Use implicit login for JCE authentication**: Whenever possible, use implicit login for JCE authentication to allow automatic credential management and re-authentication.  

- **Avoid sharing sessions across threads**: For multi-threaded applications, assign each thread its own session to prevent conflicts and security issues.  

- **Implement client-side retries**: Add retry logic for HSM operations to handle potential maintenance events or HSM replacements.  

- **Manage HSM client sessions carefully**: Be aware that `azurecloudhsm_client` shares sessions across applications on the same host, requiring proper session management to avoid conflicts.  

## Monitoring and logging

- **Monitor audit and operations logs**: Microsoft recommends configuring operation event logging. Operation event logging is vital for HSM security, providing an immutable record of access and operations for accountability, traceability, and regulatory compliance. It helps detect unauthorized access, investigate incidents, and identify anomalies, ensuring the integrity and confidentiality of cryptographic operations.  

    To maintain security and privacy, logs exclude sensitive data (e.g., key IDs, key names, user details). They capture HSM operations, timestamps, and metadata but can't determine success or failure. It can only log the fact that the operation was executed. This limitation exists because the HSM operation occurs within the inner TLS channel, which is not exposed outside that boundary.  

## Business continuity and disaster recovery  

- **Deploy disaster recovery tactics**: Microsoft Azure Cloud HSM provides high availability by clustering HSMs, synchronizing keys/policies, and auto-migrating partitions in the event of failure. Periodic backups support recovery, but BCDR best practices recommend deploying in two regions for failover. Use `extractMaskedObject` to extract keys as encrypted blobs, store them securely, and back up your HSM regularly. In case of a regional outage, restore the latest backup available to you in another region and import the maskedObjects (`insertMaskedObject`) as needed to ensure business continuity.  

## Next steps

- [Security best practices for IaaS workloads in Azure](/azure/security/fundamentals/iaas)
- [Enable just-in-time access to Virtual Machines](/azure/defender-for-cloud/just-in-time-access-usage)
- [Adopt a Zero Trust approach](/azure/security/fundamentals/network-best-practices#adopt-a-zero-trust-approach)
